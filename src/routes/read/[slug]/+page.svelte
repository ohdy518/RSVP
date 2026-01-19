<script>
    import {browser} from "$app/environment";
    import {page} from '$app/state';

    let slug = page.params.slug
    let text
    console.log(slug)
    async function getText() {

        try {
            return await import(`$lib/texts/${slug}.js`)
        } catch (e) {
            console.error(e)
        }
    }

    let wordsArray
    let interval
    let punctuationDelay

    if (browser) {
        getText().then((r) => {
            text = r.TEXT;
            interval = r.INTERVAL
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
    }

    function goCall() {
        let word = wordsArray[index]
        if ([",", ".", ";", ":", "?", "!"].includes(word.at(-1))) {
            // console.log("true!")
            extraDelay = punctuationDelay
        }
        splitSet(wordsArray[index])
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
</script>

<div class="w-screen h-screen bg-zinc-900 text-zinc-50">
    <div class="absolute left-1/2 top-1/2 -translate-x-1/2 -translate-y-1/2">
        <div
                id="texts"
                class="grid grid-cols-[1fr_auto_1fr] items-baseline gel text-4xl md:text-5xl tracking-wide whitespace-nowrap"
        >
            <span id="before" class="justify-self-end text-right">{before}</span>
            <span id="focal" class="text-rose-400">{focal}</span>
            <span id="after" class="justify-self-start text-left">{after}</span>
        </div>
        <div id="start-button" class="flex flex-col gap-1" hidden>
            <button class="inter text-3xl border-2 m-auto p-2 rounded-xl text-rose-400 font-medium" onclick={begin}>-> Go</button>
            <span class="sm:hidden inter text-lg">Rotate your phone.</span>
        </div>
    </div>
</div>