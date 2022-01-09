<script lang="ts">
    import { onMount, onDestroy } from "svelte";

    function clamp(num: number, min: number, max: number) {
        return Math.min(Math.max(num, min), max);
    }

    let drag = {
        dragging: false,
        scrollTop: 0,
        lastPageY: 0,

        start(scrollTop: number, pageY: number) {
            this.dragging = true;
            this.scrollTop = scrollTop;
            this.lastPageY = pageY;
        },

        stop() {
            this.dragging = false;
            this.scrollTop = 0;
            this.lastPageY = 0;
        },
    };

    let trackBounds = {
        top: 0,
        bottom: 0,
        padding: {
            top: 0,
            bottom: 0,
        },

        clientHeight() {
            return (
                this.top - this.bottom - this.padding.top - this.padding.bottom
            );
        },

        contains(y: number) {
            return y >= this.top && y <= this.bottom;
        },

        proportionOfHeight(y: number) {
            const ratio = (this.top - y) / this.clientHeight();
            return clamp(ratio, 0, 1);
        },

        updateFromBoundingRect(rect: DOMRect) {
            this.top = rect.top;
            this.bottom = rect.bottom;
        },

        setPadding(top: number, bottom: number) {
            this.padding.top = top;
            this.padding.bottom = bottom;
        },
    };

    let ratio = 0;
    let scrollRatio = 0;

    let wrap: HTMLDivElement;
    let content: HTMLDivElement;
    let track: HTMLDivElement;
    let bar: HTMLDivElement;

    onMount(() => {
        setupListeners();
        setTimeout(onScrollResize);
    });

    onDestroy(removeListeners);

    const resizeObserver = new ResizeObserver(onScrollResize);
    function setupListeners() {
        window.addEventListener("resize", onScrollResize);

        // check if the size of #content has changed
        for (let child of content.children) {
            resizeObserver.observe(child);
        }
    }

    function removeListeners() {
        window.removeEventListener("resize", onScrollResize);
        resizeObserver.disconnect();

        if (drag.dragging) {
            bar.removeEventListener("pointermove", onDrag);
            bar.removeEventListener("pointerup", onStopDrag);
        }
    }

    function onStartDrag(e: Event) {
        const pointerEvent = <PointerEvent>e;

        bar.classList.add("grabbed");
        bar.addEventListener("pointermove", onDrag);
        bar.addEventListener("pointerup", onStopDrag);
        bar.setPointerCapture(pointerEvent.pointerId);

        drag.start(content.scrollTop, pointerEvent.pageY);
        onDrag(pointerEvent);
    }

    function onStopDrag(e: Event) {
        const pointerEvent = <PointerEvent>e;

        bar.classList.remove("grabbed");
        bar.removeEventListener("pointermove", onDrag);
        bar.removeEventListener("pointerup", onStopDrag);

        bar.releasePointerCapture(pointerEvent.pointerId);
        setTimeout(drag.stop.bind(drag));
    }

    function onDrag(e: Event) {
        const pointerEvent = <PointerEvent>e;

        const pageY = pointerEvent.pageY;
        const delta = pageY - drag.lastPageY;

        requestAnimationFrame(() => {
            const docScrollTop = document.documentElement.scrollTop;
            if (trackBounds.contains(pageY - docScrollTop)) {
                // drag is within the bounds of the track
                content.scrollTop = drag.scrollTop + delta / ratio;
            } else {
                drag.scrollTop = content.scrollTop;
                drag.lastPageY = pointerEvent.pageY;
            }
        });
    }

    function onTrackClick(e: Event) {
        if (drag.dragging) return;
        const pointerEvent = <PointerEvent>e;

        const pageRatio = wrap.clientHeight / content.scrollHeight;
        const pointerRatio = trackBounds.proportionOfHeight(
            pointerEvent.clientY
        );
        const newRatio = clamp(
            clamp(
                pointerRatio,
                scrollRatio - pageRatio,
                scrollRatio + pageRatio
            ),
            0,
            1 - pageRatio
        );

        content.scrollTop =
            newRatio * (content.scrollHeight - wrap.clientHeight);
        updateBar();
    }

    let resizeDebouceTimeout: number | undefined = undefined;
    function onScrollResize() {
        ratio = track.clientHeight / content.scrollHeight;
        updateBar();

        clearTimeout(resizeDebouceTimeout);
        resizeDebouceTimeout = setTimeout(recalculateTrackBounds, 10);
    }

    function recalculateTrackBounds() {
        const boundingRect = track.getBoundingClientRect();
        trackBounds.updateFromBoundingRect(boundingRect);

        const trackStyle = window.getComputedStyle(track, null);
        trackBounds.setPadding(
            parseInt(trackStyle.paddingTop),
            parseInt(trackStyle.paddingBottom)
        );

        setTimeout(() => {
            track.style.display =
                wrap.clientHeight < content.scrollHeight ? "block" : "none";
        });
    }

    function updateBar() {
        const totalHeight = content.scrollHeight;
        const clientHeight = wrap.clientHeight;

        scrollRatio = content.scrollTop / (totalHeight - clientHeight);

        requestAnimationFrame(() => {
            const barHeight = (clientHeight / totalHeight) * 100;
            const barTop = (content.scrollTop / totalHeight) * 100;

            bar.style.setProperty("height", `${barHeight}%`);
            bar.style.setProperty("top", `${barTop}%`);
        });
    }
</script>

<div id="outer" on:pointerenter="{onScrollResize}">
    <div id="wrap" bind:this="{wrap}">
        <div
            id="content"
            bind:this="{content}"
            on:resize="{onScrollResize}"
            on:scroll="{onScrollResize}"
        >
            <slot />
        </div>
    </div>
    <div id="track" bind:this="{track}" on:click="{onTrackClick}">
        <div id="bar" bind:this="{bar}" on:pointerdown="{onStartDrag}"></div>
    </div>
</div>

<style>
    #outer {
        overflow-y: visible !important;
        height: 100%;
    }

    #wrap {
        overflow: hidden;
        height: 100%;
        position: relative;
        z-index: 1;
        margin-right: 14px;
    }

    #content {
        height: 100%;
        width: 100%;
        position: relative;
        overflow-y: auto;
        overflow-x: hidden;
        -moz-box-sizing: border-box;
        box-sizing: border-box;
        scroll-behavior: unset;

        scrollbar-width: none;
    }

    #content::-webkit-scrollbar {
        display: none;
    }

    #track {
        position: absolute;
        right: 4px;
        border-radius: 4px;
        transition: background 0.2s;
        top: 4px;
        bottom: 4px;
        width: 10px;
        cursor: default;
    }

    #track:hover {
        background: rgb(0, 0, 0, 0.2);
        transition: background 0.2s;
    }

    #bar {
        position: relative;
        background: rgb(0, 0, 0, 0.6);
        width: 100%;
        border-radius: 4px;
        right: 0;
        top: 0;
        z-index: 0;
        transition: background 0.1s;
        cursor: -moz-grab;
        cursor: -webkit-grab;
    }

    #bar:hover {
        background: rgb(0, 0, 0, 0.4);
    }

    #track :global(#bar.grabbed),
    #track :global(#bar.grabbed:hover) {
        cursor: -moz-grabbing;
        cursor: -webkit-grabbing;
        touch-action: none;
    }

    :global(body) {
        overflow-x: hidden;
    }
</style>
