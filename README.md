<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Lua Obfuscator (Web Safe)</title>
<style>
    body {
        background:#0f0f0f;
        color:#fff;
        font-family: monospace;
        margin:0;
        padding:20px;
    }
    textarea {
        width:100%;
        height:200px;
        background:#1a1a1a;
        color:#0f0;
        border:1px solid #333;
        padding:10px;
        font-size:14px;
        resize:vertical;
    }
    button {
        background:#222;
        color:#fff;
        border:1px solid #444;
        padding:10px 15px;
        margin-top:10px;
        cursor:pointer;
    }
    button:hover { background:#333; }
    .row { margin-bottom:15px; }
    label { display:block; margin-bottom:5px; color:#ccc; }
</style>
</head>

<body>

<h2>Lua Obfuscator (Browser / Safe)</h2>
<p style="color:#aaa">
This tool <b>only generates text</b>.  
It does NOT execute Lua and will NOT crash Roblox.
</p>

<div class="row">
<label>Lua Input</label>
<textarea id="input">print("hi")</textarea>
</div>

<div class="row">
<label>Target Size (characters)</label>
<input id="size" type="number" value="20000" style="width:100px">
</div>

<button onclick="obfuscate()">Obfuscate</button>
<button onclick="copyOutput()">Copy Output</button>

<div class="row">
<label>Obfuscated Output</label>
<textarea id="output"></textarea>
</div>

<script>
function toDecEsc(str) {
    let out = [];
    for (let i = 0; i < str.length; i++) {
        out.push("\\" + str.charCodeAt(i));
    }
    return out.join("");
}

function junkBlock() {
    return `
local _j = "${Math.random().toString(36).repeat(5)}"
for _=1,30 do
    _j = _j:sub(1,#_j)
end
`;
}

function wrapLayer(code) {
    const encoded = toDecEsc(code);
    return `
return(function(...)
${junkBlock()}
local function _D(z)
    local o = {}
    for n in tostring(z):gmatch("%d+") do
        o[#o+1] = string.char(tonumber(n))
    end
    return table.concat(o)
end
(${("_D")})("${encoded}")()
${junkBlock()}
end)(...)
`;
}

function obfuscate() {
    let code = document.getElementById("input").value;
    let target = parseInt(document.getElementById("size").value) || 20000;

    let result = code;

    // Safe number of layers (browser-friendly)
    for (let i = 0; i < 3; i++) {
        result = wrapLayer(result);
    }

    // Pad size without deep recursion
    while (result.length < target) {
        result += junkBlock();
    }

    document.getElementById("output").value =
`--[[ Web Lua Obfuscator v1.0 (Safe) ]]
${result}`;
}

function copyOutput() {
    const out = document.getElementById("output");
    out.select();
    document.execCommand("copy");
    alert("Copied to clipboard!");
}
</script>

</body>
</html>
