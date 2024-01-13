<script lang="js">    
    let nowPage = 0;

    let rawInput = '';
    let prettierResult = '';
    let path = [];
    let valueArr = [];
    let parsedJsonArr = [];
    let chosenIdx = -1;

    let isInvalid = false;
    let parsedJson = {};

    function prettier() {
        try {
            parsedJson = JSON.parse(rawInput);
            findValuePath(parsedJson, 0);
            parsedJsonArr = prettierResult.split('\n');
            nowPage = 1;
        } catch {
            isInvalid = true;
            alert('JSON parsing fail\nPlease check your input.');
        }

    }

    function findValuePath(jsonObj, depth, beforeIsArr) {
        if (isInvalid) return;

        if (jsonObj === null) {
            prettierResult += `${' '.repeat(4*(depth))}@#${valueArr.length}@`;
            valueArr.push({value: 'null', path: path.slice()});
            return;
        } else if (typeof jsonObj === 'string') {
            prettierResult += `${' '.repeat(4*(depth))}@#${valueArr.length}@`;
            valueArr.push({value: `"${jsonObj}"`, path: path.slice()});
            return;
        } else if (typeof jsonObj !== 'object') {
            prettierResult += `${' '.repeat(4*(depth))}@#${valueArr.length}@`;
            valueArr.push({value: jsonObj.toString, path: path.slice()});
            return;
        }

        prettierResult += beforeIsArr ? ' '.repeat(4*depth) + '{\n' : '{\n';
        valueArr.push({value: null, path: null});
        
        let objLength = Object.keys(jsonObj).length;
        let idx = 0;
        for (let key in jsonObj) {
            idx += 1;
            path.push(key);
            prettierResult += `${' '.repeat(4*(depth+1))}"${key}": `;
            if (Array.isArray(jsonObj[key])) {
                prettierResult += '[\n';
                let jdx = 0;
                let subLength = jsonObj[key].length;
                for (let i in jsonObj[key]) {
                    jdx += 1;
                    path.push(Number(i));
                    findValuePath(jsonObj[key][i], depth+2, true);
                    prettierResult += jdx !== subLength ? ',\n' : '\n';
                    path.pop();
                }
                prettierResult += ' '.repeat(4*(depth+1)) + ']';
            } else if (jsonObj[key] != null && typeof jsonObj[key] === 'object') {
                findValuePath(jsonObj[key], depth+1, false);
            } else if (jsonObj[key] == null) {
                prettierResult += `@#${valueArr.length}@`;
                valueArr.push({value: 'null', path: path.slice()});
            } else {
                prettierResult += `@#${valueArr.length}@`;
                if (typeof jsonObj[key] === 'string') valueArr.push({value: `"${jsonObj[key].toString()}"`, path: path.slice()});
                else valueArr.push({value: `${jsonObj[key].toString()}`, path: path.slice()});
            }
            
            prettierResult += idx !== objLength ?  ',\n' : '\n';
            path.pop();
        }

        prettierResult += ' '.repeat(4*depth) + '}';
        
    }

    function changeChosenValue(idx) {
        chosenIdx = idx;
        console.log(chosenIdx);
    }

    function buildPathStr(idx) {
        let pathStr = 'target = json_element';
        for (let i in valueArr[idx].path) {
            if (typeof valueArr[idx].path[i] === 'string') {
                pathStr += `["${valueArr[idx].path[i]}"]`;
            } else pathStr += `[${valueArr[idx].path[i].toString()}]`;
        }

        return pathStr;
    }

    function copyPrettierJson() {
        window.navigator.clipboard.writeText(prettierResult).then(() => {
            alert('JSON이 복사되었습니다.');
        });
    }
</script>

<svelte:head>
	<title>JSON 경로 탐색기</title>
    <meta name="description" content="크롤링 등의 일을 하면서 필요한 JSON 내의 특정 정보의 경로를 빠르게 찾아드립니다">
    <meta name="keywords" content="JSON, 크롤링, Crawling, 경로, 경로탐색, 경로계산" />
    <meta property="og:title" content="JSON 경로 탐색기">
    <meta property="og:type" content="website">
</svelte:head>

<div class="wrapper">
    <h1>JSON 경로 탐색기</h1>
    <p>크롤링 등의 일을 하면서 필요한 JSON 내의 특정 정보의 경로를 빠르게 찾아드립니다.</p>
    <div class="inner-wrapper">
        {#if nowPage == 0}
            <textarea bind:value={rawInput} class="user-input" placeholder="분석할 JSON text를 입력하세요"></textarea>
            <button on:click={prettier} class="prettier-btn">경로 계산</button>
        {:else}
            <div class="prettier-result">
                {#each parsedJsonArr as line}
                    <div>
                        {#each line.replaceAll(' ', '\u00A0').split('@') as item}
                            {#if item.slice(0, 1) === '#'}
                                <button class="{chosenIdx === Number(item.slice(1)) ? 'selected' : ''}" on:click={() => changeChosenValue(Number(item.slice(1)))}>{valueArr[Number(item.slice(1))].value}</button>
                            {:else}
                                <span>{item}</span>
                            {/if}
                        {/each}
                    </div>
                {/each}    
            </div>
            <div class="path">{chosenIdx !== -1 ? buildPathStr(chosenIdx) : ''}</div>
        {/if}
    </div>
</div>

<style>
    @font-face {
        font-family: 'D2Coding';
        src: url('https://cdn.jsdelivr.net/gh/projectnoonnu/noonfonts_three@1.0/D2Coding.woff') format('woff');
        font-weight: normal;
        font-style: normal;
    }

    @font-face {
        font-family: 'S-CoreDream-3Light';
        src: url('https://cdn.jsdelivr.net/gh/projectnoonnu/noonfonts_six@1.2/S-CoreDream-3Light.woff') format('woff');
        font-weight: normal;
        font-style: normal;
    }

    h1 {
        margin-top: 50px;
        font-family: 'S-CoreDream-3Light';
        font-weight: 800;
    }

    p {
        font-family: 'S-CoreDream-3Light';
    }

    .wrapper {
        width: 100vw;
        /* height: 100vh; */
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
    }

    .inner-wrapper {
        display: flex;
        flex-direction: column;
        justify-content: space-between;
        align-items: center;
    }

    textarea {
        width: 60vw;
        height: 60vh;
        resize: none;
        font-size: 20px;
        font-family: "D2Coding";
        outline: none;
        border: 1.5px solid #DDDDDD;
        padding: 20px;
    }

    .prettier-result {
        width: 60vw;
        height: 60vh;
        font-size: 20px;
        font-size: 20px;
        font-family: "D2Coding";
        border: 1.5px solid #DDDDDD;
        padding: 20px;
        overflow: auto;
    }

    .selected {
        background-color: #d3d3d3;
    }

    button {
        border: 1px solid #DDDDDD;
        background-color: #ececec;
        padding: 5px;
        cursor: pointer;
    }

    .prettier-btn {
        background-color: #1D1D1D;
        color: white;
        width: 170px;
        font-size: 17px;
        height: 40px;
        font-weight: bold;
        margin-top: 15px;
        cursor: pointer;
    }

    .path {
        font-family: "D2Coding";
        font-size: 20px;
        padding: 10px;
        background-color: #ececec;
        margin-top: 20px;
    }
</style>