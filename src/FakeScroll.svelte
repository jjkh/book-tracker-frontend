<script>
    import { onMount } from "svelte";

    const clamp = (num, min, max) => Math.min(Math.max(num, min), max);

    // TODO: extract this into it's own .ts file
    let drag = {
        dragging: false,
        scrollTop: 0,
        lastPageY: 0,

        start(scrollTop, pageY) {
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

        contains(y) {
            return y >= this.top && y <= this.bottom;
        },

        proportionOfHeight(y) {
            const ratio = (y - this.top) / this.clientHeight();
            return clamp(ratio, 0, 1);
        },

        updateFromBoundingRect(rect) {
            this.top = rect.top;
            this.bottom = rect.bottom;
        },

        setPadding(top, bottom) {
            this.padding.top = top;
            this.padding.bottom = bottom;
        },
    };

    let ratio = 0;
    let scrollRatio = 0;

    let outer = undefined;
    let wrap = undefined;
    let content = undefined;
    let track = undefined;
    let bar = undefined;

    onMount(() => {
        outer = document.getElementById("outer");
        wrap = document.getElementById("wrap");
        content = document.getElementById("content");
        track = document.getElementById("track");
        bar = document.getElementById("bar");

        setupCallbacks();
        setTimeout(onScrollResize());
    });

    function setupCallbacks() {
        window.addEventListener("resize", onScrollResize);
        content.addEventListener("resize", onScrollResize);
        content.addEventListener("scroll", onScrollResize);
        outer.addEventListener("mouseenter", onScrollResize);
        // TODO: fix this
        // track.addEventListener("click", onTrackClick);
        bar.addEventListener("mousedown", onStartDrag);
    }

    // TODO: rework these event listeners to be less bad
    function onStartDrag(mouseEvent) {
        drag.start(content.scrollTop, mouseEvent.pageY);

        document.body.classList.add("grabbed");
        bar.classList.add("grabbed");
        document.body.addEventListener("mousemove", onDrag);
        bar.addEventListener("mousemove", onDrag);
        document.body.addEventListener("mouseup", onStopDrag);
        bar.addEventListener("mouseup", onStopDrag);

        onDrag(mouseEvent);
    }

    function onStopDrag(_mouseEvent) {
        document.body.classList.remove("grabbed");
        bar.classList.remove("grabbed");
        document.body.removeEventListener("mousemove", onDrag);
        bar.removeEventListener("mousemove", onDrag);
        document.body.removeEventListener("mouseup", onStopDrag);
        bar.removeEventListener("mouseup", onStopDrag);

        drag.stop();
    }

    function onDrag(mouseEvent) {
        const pageY = mouseEvent.pageY;
        const delta = pageY - drag.lastPageY;

        requestAnimationFrame(() => {
            const docScrollTop = document.documentElement.scrollTop;
            // NOTE: unsure what this is even doing?
            if (trackBounds.contains(pageY - docScrollTop)) {
                // drag is within the bounds of the track
                content.scrollTop = drag.scrollTop + delta / ratio;
            } else {
                drag.scrollTop = content.scrollTop;
                drag.lastPageY = mouseEvent.pageY;
            }
        });
    }

    function onTrackClick(mouseEvent) {
        if (drag.dragging) return;

        const pageRatio = wrap.clientHeight / content.scrollHeight; // should be less padding?
        const mouseRatio = trackBounds.proportionOfHeight(mouseEvent.clientY);
        const newRatio = clamp(
            clamp(mouseRatio, scrollRatio - pageRatio, scrollRatio + pageRatio),
            0,
            1 - pageRatio
        );

        content.scrollTop = newRatio * (content.scrollHeight - wrap.height);
        updateBar();
    }

    function onScrollResize() {
        ratio = track.clientHeight / content.scrollHeight;
        updateBar();

        // possibly need to update scrollbar visibility based on ratio here, but doesn't quite seem right

        // may need to "debounce" bounds recalculation(?)
        const boundingRect = track.getBoundingClientRect();
        trackBounds.updateFromBoundingRect(boundingRect);

        const trackStyle = window.getComputedStyle(track, null);
        trackBounds.setPadding(trackStyle.paddingTop, trackStyle.paddingBottom);
    }

    function updateBar() {
        const totalHeight = content.scrollHeight;
        const clientHeight = wrap.clientHeight;

        scrollRatio = content.scrollTop / (totalHeight - clientHeight);

        requestAnimationFrame(() => {
            const barHeight = (clientHeight / totalHeight) * 100;
            const barTop = (content.scrollTop / totalHeight) * 100;

            // seems gross - there must be a better way!
            bar.style.cssText = `height: ${barHeight}%; top: ${barTop}%;`;
        });
    }
</script>

<div id="outer">
    <div id="wrap">
        <div id="content">
            <slot />
        </div>
    </div>
    <div id="track">
        <div id="bar"></div>
    </div>
</div>

<style>
    #outer {
        overflow: visible !important;
        height: 100%;
    }

    #wrap {
        overflow: hidden;
        height: 100%;
        position: relative;
        z-index: 1;
    }

    #content {
        height: 100%;
        width: 100%;
        position: relative;
        overflow: auto;
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
        right: -15px;
        top: 0;
        bottom: 0;
        width: 9px;
        cursor: default;
    }

    #bar {
        position: relative;
        background: rgba(0, 255, 255, 0.4);
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
        background: rgba(255, 0, 255, 0.55);
    }

    #bar.grabbed {
        cursor: -moz-grabbing;
        cursor: -webkit-grabbing;
        background: green;
    }

    :global(body.grabbed) {
        cursor: -moz-grabbing;
        cursor: -webkit-grabbing;
        -moz-user-select: none;
        -webkit-user-select: none;
        user-select: none;
    }
</style>
