<script>
    import { onMount } from "svelte";

    // count must be a square number otherwise it will be rounded up to the next square number
    export let count = 1;
    export let initialStrokes = 1000;
    let canvas;
    let ctx = null;
    let clear = false;

    const step = 5;
    let pos = [];

    const drawRandomStroke = (i) => {
        let randAngle = Math.random() * 2 * Math.PI;
        pos[i][0] += Math.sin(randAngle) * step;
        pos[i][1] += Math.cos(randAngle) * step;

        pos[i][0] = pos[i][0] < 0 ? 0 : pos[i][0];
        pos[i][1] = pos[i][1] < 0 ? 0 : pos[i][1];

        pos[i][0] = pos[i][0] > canvas.width ? canvas.width : pos[i][0];
        pos[i][1] = pos[i][1] > canvas.height ? canvas.height : pos[i][1];

        ctx.lineTo(pos[i][0], pos[i][1]);
    };

    const addSegment = () => {
        clear ||= Math.abs(window.innerWidth*window.innerHeight / (canvas.width*canvas.height) - 1) > 0.2 ;
        clear ||= Math.abs(window.innerWidth/window.innerHeight - canvas.width/canvas.height) > 0.5 ;

        if (clear) {
            canvas.width = window.innerWidth;
            canvas.height = window.innerHeight;
            ctx = canvas.getContext("2d");
            ctx.strokeStyle = "rgba(0, 0, 0, 0.2)";

            // initial strokes
            pos = [];
            let points_row_col = Math.ceil(Math.sqrt(count));
            count = points_row_col * points_row_col;
            for (let i = 0; i < count; i++) {
                pos.push([
                    ((i % points_row_col) * canvas.width) / points_row_col,
                    (Math.floor(i / points_row_col) * canvas.height) /
                        points_row_col,
                ]);

                pos[i][0] += (Math.random() * canvas.width) / points_row_col;
                pos[i][1] += (Math.random() * canvas.height) / points_row_col;

                ctx.beginPath();
                ctx.moveTo(pos[i][0], pos[i][1]);
                for (let j = 0; j < initialStrokes; j++) {
                    drawRandomStroke(i);
                }
                ctx.stroke();
            }
            clear = false;
        }

        for (let i = 0; i < pos.length; i++) {
            ctx.beginPath();
            ctx.moveTo(pos[i][0], pos[i][1]);
            drawRandomStroke(i);
            ctx.stroke();
        }

        setTimeout(addSegment, 10);
    };

    onMount(async () => {
        clear = true;
        addSegment();
    });
</script>

<canvas bind:this={canvas}></canvas>

<style>
    canvas {
        position: fixed;
        top: 0;
        left: 0;
        width: 100vw;
        height: 100vh;
        z-index: -100;
    }
</style>
