<script>
    import {browser} from "$app/environment";
    import {page} from '$app/state';

    let slug = page.params.slug
    let text
    let focalElement;
    let focalWidth = $state(0);
    let root;
    console.log(slug)
    async function getText() {

        try {
            return await import(`$lib/texts/dmt.js`)
        } catch (e) {
            console.error(e)
        }
    }

    let wordsArray
    let interval
    let punctuationDelay

    if (browser) {
        root = document.querySelector(':root');
        root.style.setProperty("--desired-width", `${3}px`);
        getText().then((r) => {
            text = r.TEXT;
            interval = 300
            punctuationDelay = r.PUNCTUATION_DELAY
            wordsArray = text.split(" ");
            document.getElementById('start-button').hidden = false;
        })
    }

    let index = 0

    let before = $state("")
    let focal = $state("")
    let after = $state("")


    let extraDelay = 0


    let stopCall = false

    function splitSet(word) {
        switch (word.length) {
            case 0:
                before = "";
                focal = "";
                after = "";
                break;
            case 1:
                before = "";
                focal = word;
                after = "";
                break;

            case 2:
            case 3:
            case 4:
            case 5:
                before = word[0] ?? "";
                focal = word[1] ?? "";
                after = word.slice(2); // from index 2 to end
                break;

            default:
                before = word.slice(0, 2); // first 2 chars (0 and 1)
                focal = word[2] ?? "";
                after = word.slice(3); // from index 3 to end
                break;
        }
        root.style.setProperty("--desired-width", `${getTextWidth(focal.trim(), getCanvasFont(focalElement))+1.2}px`);
    }

    function goCall() {
        let word = wordsArray[index]
        if ([",", ".", ";", ":", "?", "!"].includes(word.at(-1))) {
            // console.log("true!")
            extraDelay = punctuationDelay
        }

        splitSet(wordsArray[index])
        focalWidth = focalElement.offsetWidth
        // console.log(getTextWidth("i", getCanvasFont(focalElement))+0.75);
        // // console.log(focalWidth)
        // root.style.setProperty("--desired-width", `${focalElement.offsetWidth}px`);
        // console.log(getComputedStyle(root).getPropertyValue("--desired-width"));
        // console.log(focalElement.offsetWidth)
        index++;
        if ((index) >= wordsArray.length) {
            stopCall = true;
        }
    }

    function go() {
        if (stopCall) {
            splitSet("")
            return
        }

        const delay = interval + extraDelay;
        extraDelay = 0;

        setTimeout(() => {
            goCall();
            go(); // schedule next tick
        }, delay);
    }

    function begin() {
        document.getElementById('start-button').hidden = true;
        extraDelay = 300;
        go()
    }


    function getTextWidth(text, font) {
        // re-use canvas object for better performance
        const canvas = getTextWidth.canvas || (getTextWidth.canvas = document.createElement("canvas"));
        const context = canvas.getContext("2d");
        context.font = font;
        const metrics = context.measureText(text);
        return metrics.width;
    }

    function getCssStyle(element, prop) {
        return window.getComputedStyle(element, null).getPropertyValue(prop);
    }

    function getCanvasFont(el = document.body) {
        const fontWeight = getCssStyle(el, 'font-weight') || 'normal';
        const fontSize = getCssStyle(el, 'font-size') || '16px';
        const fontFamily = getCssStyle(el, 'font-family') || 'Times New Roman';

        return `${fontWeight} ${fontSize} ${fontFamily}`;
    }
</script>

<div class="w-screen h-screen bg-zinc-900 text-zinc-50">
    <div class="absolute left-1/2 top-1/2 -translate-x-1/2 -translate-y-1/2">
        <div
                id="texts"
                class="text-grid items-baseline text-3xl md:text-5xl gel tracking-wide whitespace-nowrap"
        >
            <span id="before" class="justify-self-end text-right">{before}</span>
            <span id="focal" bind:this={focalElement} class="text-rose-400 text-3xl md:text-5xl gel">{focal}</span>
            <span id="after" class="justify-self-start text-left">{after}</span>
        </div>
        <div id="start-button" class="flex flex-col gap-1" hidden>
            <button class="inter text-3xl border-2 m-auto p-2 rounded-xl text-rose-400 font-medium" onclick={begin}>-> Go</button>
            <span class="sm:hidden inter text-lg">Rotate your phone.</span>
        </div>
<!--        <span>{focalWidth}</span>-->
    </div>
</div>