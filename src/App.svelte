<script>
    import {onMount} from "svelte";

    let selectorExpr = '';
    let result = '';

    let count = 0;

    function queryAndJoin(selector) {
        const selectedElements = document.querySelectorAll(selector);
        let resultStr = [...selectedElements].map(a => a.attributes.href.nodeValue).join("\r\n\r\n");

        chrome.runtime.sendMessage({
            action: 'result',
            result: resultStr,
            count: selectedElements.length
        }, function (response) {
            // chrome.tts.speak(`${selectedElements.length} elements are found and returned.`);
        })
    }

    function onKeydown(e) {
        if (e.key == 'Enter') {
            evalExpr()
        }
    }

    function evalExpr() {
        chrome.tabs.query({active: true, currentWindow: true}, function (tabs) {
            chrome.scripting.executeScript({
                target: {tabId: tabs[0].id},
                function: queryAndJoin,
                args: [selectorExpr],
            });
        });
    }

    function copyToClipboard() {
        navigator.clipboard.writeText(result).then(() => {
            //clipboard successfully set
        }, () => {
            //clipboard write failed, use fallback
            copyTextToClipboard(result)
        });
    }

    function copyTextToClipboard(text) {
        //Create a textbox field where we can insert text to.
        let copyFrom = document.createElement("textarea");

        //Set the text content to be the text you wished to copy.
        copyFrom.textContent = text;

        //Append the textbox field into the body as a child.
        //"execCommand()" only works when there exists selected text, and the text is inside
        //document.body (meaning the text is part of a valid rendered HTML element).
        document.body.appendChild(copyFrom);

        //Select all the text!
        copyFrom.select();

        //Execute command
        document.execCommand('copy');

        //(Optional) De-select the text using blur().
        copyFrom.blur();

        //Remove the textbox field from the document.body, so no other JavaScript nor
        //other elements can get access to this.
        document.body.removeChild(copyFrom);
    }

    onMount(async () => {
        chrome.runtime.onMessage.addListener(function (request, sender, sendResponse) {
            if (request.action === "result") {
                result = request.result;
                count = request.count;
                chrome.tts.speak(`${request.count} elements are found and returned.`);
            } else {
                console.log(`not supported action: ${request.action}`)
            }
        });
    })

</script>

<section class="text-gray-600 body-font relative">
    <div class="container px-5 py-5 mx-auto">
        <div class="w-full text-center mb-6">
            <h1 class="text-3xl font-extrabold text-primary">Select'n'Copy</h1>
        </div>
        <div class="lg:w-1/2 md:w-2/3 mx-auto">
            <div class="flex flex-wrap -m-2">
                <div class="p-2 w-full">
                    <div class="form-control w-full">
                        <label class="label" for="selectorExpr">
                            <span class="label-text">Selector Expression</span>
                        </label>
                        <!-- svelte-ignore a11y-autofocus -->
                        <input id="selectorExpr" type="text" placeholder="Type selector expression here"
                               class="input input-bordered input-sm w-full"
                               bind:value={selectorExpr} on:keydown={onKeydown} autofocus/>
                    </div>

                </div>
                <div class="p-2 w-full inline-flex items-baseline">
                    <button class="btn btn-block btn-primary" on:click={evalExpr}>Evaluate</button>
                </div>

                <div class="p-2 w-full">
                    <div class="form-control w-full">
                        <label class="label" for="result">
                            <span class="label-text">Result Content</span>
                        </label>
                        <!-- svelte-ignore a11y-autofocus -->
                        <textarea id="result"
                                  class="textarea textarea-bordered h-36 " bind:value={result}/>
                        <span>
                            {count} elements are found before joined as string
                        </span>
                    </div>
                </div>

                <div class="p-2 w-full inline-flex items-baseline">
                    <button class="btn btn-block" on:click={copyToClipboard}>Copy to Clipboard</button>
                </div>
            </div>
        </div>

    </div>


</section>


<style lang="postcss" global>
    @tailwind base;
    @tailwind components;
    @tailwind utilities;
</style>