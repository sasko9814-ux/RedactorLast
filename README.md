<!DOCTYPE html>
<html lang="uk">
<head>
<meta charset="UTF-8">
<title>Редактор коду з контекстними підказками</title>

<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/codemirror/5.65.14/codemirror.min.css">
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/codemirror/5.65.14/theme/dracula.min.css">
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/codemirror/5.65.14/theme/eclipse.min.css">
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/codemirror/5.65.14/addon/hint/show-hint.min.css">

<style>
*{box-sizing:border-box;margin:0;padding:0;}
body{display:flex;height:100vh;font-family:Segoe UI,sans-serif;background:#1e1e1e;color:#fff;}
#editor-container{width:50%;display:flex;flex-direction:column;border-right:2px solid #333;}
#editor-toolbar{display:flex;gap:5px;align-items:center;padding:5px;background:#2d2d2d;border-bottom:2px solid #333;}
#editor-toolbar button{padding:5px 10px;border:none;border-radius:5px;font-weight:bold;cursor:pointer;transition:0.2s;}
#editor-toolbar button:hover{opacity:0.8;}
#download-btn{background:#28a745;color:#fff;}
#upload-btn{background:#007bff;color:#fff;}
#format-btn{background:#fd7e14;color:#fff;}
#theme-btn{background:#6f42c1;color:#fff;}
#editor-logo{height:30px;margin-right:10px;user-select:none;}
.tabs{display:flex;border-bottom:2px solid #333;}
.tab{padding:10px 20px;cursor:pointer;font-weight:bold;color:#fff;transition:0.2s;}
.tab:hover{opacity:0.8;}
.tab.active{border-bottom:3px solid #fff;}
.tab[data-lang="html"]{background:#28a745;}
.tab[data-lang="css"]{background:#007bff;}
.tab[data-lang="js"]{background:#fd7e14;}
.CodeMirror{flex:1;height:calc(100vh - 80px);}
#preview{width:50%;background:#f9f9f9;overflow:auto;}
#preview iframe{width:100%;height:100%;border:none;}
.light-theme body{background:#f0f0f0;color:#000;}
.light-theme #editor-container,.light-theme .CodeMirror{background:#fff;color:#000;}
.light-theme #editor-toolbar{background:#ddd;border-bottom-color:#aaa;}
.light-theme .tab{color:#000;}
</style>
</head>
<body>

<div id="editor-container">
  <div id="editor-toolbar">
    <img id="editor-logo" src="https://i.imgur.com/4AiXzf8.png" alt="CodeLab Logo">
    <button id="download-btn">Завантажити файл</button>
    <button id="upload-btn">Вивантажити файл</button>
    <button id="format-btn">Форматувати код</button>
    <button id="theme-btn">Тема</button>
    <input type="file" id="file-input" style="display:none;">
  </div>
  <div class="tabs">
    <div class="tab active" data-lang="html">HTML</div>
    <div class="tab" data-lang="css">CSS</div>
    <div class="tab" data-lang="js">JS</div>
  </div>
  <textarea id="code-editor"></textarea>
</div>

<div id="preview"><iframe id="live-preview"></iframe></div>

<script src="https://cdnjs.cloudflare.com/ajax/libs/codemirror/5.65.14/codemirror.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/codemirror/5.65.14/mode/xml/xml.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/codemirror/5.65.14/mode/javascript/javascript.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/codemirror/5.65.14/mode/css/css.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/codemirror/5.65.14/mode/htmlmixed/htmlmixed.min.js"></script>

<script src="https://cdnjs.cloudflare.com/ajax/libs/codemirror/5.65.14/addon/edit/closebrackets.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/codemirror/5.65.14/addon/edit/closetag.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/codemirror/5.65.14/addon/hint/show-hint.min.js"></script>

<script src="https://cdnjs.cloudflare.com/ajax/libs/js-beautify/1.15.1/beautify.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/js-beautify/1.15.1/beautify-html.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/js-beautify/1.15.1/beautify-css.min.js"></script>

<script>
const editor = CodeMirror.fromTextArea(document.getElementById('code-editor'), {
  lineNumbers: true,
  mode: "htmlmixed",
  theme: "dracula",
  autoCloseBrackets: true,
  autoCloseTags: true,
  extraKeys: {"Ctrl-Space":"autocomplete","Tab":handleTab}
});

const tabs = document.querySelectorAll('.tab');
let currentLang = 'html';
let code = {html:'', css:'', js:''};

if(localStorage.getItem('codeEditorData')){
  code = JSON.parse(localStorage.getItem('codeEditorData'));
  editor.setValue(code[currentLang]);
  updatePreview();
}

tabs.forEach(tab=>{
  tab.addEventListener('click', ()=>{
    tabs.forEach(t=>t.classList.remove('active'));
    tab.classList.add('active');
    currentLang = tab.dataset.lang;
    editor.setOption('mode', currentLang==='html'?'htmlmixed':currentLang);
    editor.setValue(code[currentLang]);
  });
});

editor.on('change', ()=>{
  code[currentLang] = editor.getValue();
  localStorage.setItem('codeEditorData', JSON.stringify(code));
  updatePreview();
});

document.getElementById('download-btn').addEventListener('click',()=>{
  const blob = new Blob([editor.getValue()], {type:'text/plain'});
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  a.download = `${currentLang}_code.txt`;
  a.click();
  URL.revokeObjectURL(url);
});

document.getElementById('upload-btn').addEventListener('click',()=>document.getElementById('file-input').click());
document.getElementById('file-input').addEventListener('change', e=>{
  const file = e.target.files[0];
  if(!file) return;
  const reader = new FileReader();
  reader.onload = function(ev){ editor.setValue(ev.target.result); }
  reader.readAsText(file);
});

document.getElementById('theme-btn').addEventListener('click', ()=>{
  document.body.classList.toggle('light-theme');
  editor.setOption('theme', document.body.classList.contains('light-theme') ? 'eclipse' : 'dracula');
});

document.getElementById('format-btn').addEventListener('click', ()=>{
  let val = editor.getValue(), formatted='';
  if(currentLang==='html') formatted = html_beautify(val, {indent_size:2});
  if(currentLang==='css') formatted = css_beautify(val, {indent_size:2});
  if(currentLang==='js') formatted = js_beautify(val, {indent_size:2});
  editor.setValue(formatted);
});

/* HTML теги за контекстом */
const htmlTags = {
  head: ['base','link','meta','noscript','script','style','title'],
  body: ['header','nav','main','footer','section','article','aside','h1','h2','h3','p','div','span','ul','li','a','img','form','input','button','video','audio'],
  nav: ['a','ul','li','div','span'],
  main: ['section','article','h1','h2','p','div','span','img','form'],
  footer: ['p','div','span','a','ul','li'],
  form: ['input','textarea','button','select','label','fieldset'],
};

/* CSS та JS підказки */
const cssHints = [
  {text:'color'}, {text:'background-color'}, {text:'margin'}, {text:'padding'}, {text:'font-size'}, {text:'font-family'},
  {text:'display'}, {text:'position'}, {text:'width'}, {text:'height'}, {text:'border'}, {text:'border-radius'},
  {text:'overflow'}, {text:'text-align'}, {text:'justify-content'}, {text:'align-items'}, {text:'flex'}, {text:'grid'},
  {text:'opacity'}, {text:'z-index'}, {text:'display: flex'}, {text:'display: grid'}, {text:'position: absolute'}, {text:'position: relative'}
];
const jsHints = [
  {text:'function'}, {text:'return'}, {text:'var'}, {text:'let'}, {text:'const'}, {text:'if'}, {text:'else'}, {text:'for'},
  {text:'while'}, {text:'console.log()'}, {text:'document.getElementById()'}, {text:'document.querySelector()'},
  {text:'addEventListener'}, {text:'setTimeout'}, {text:'setInterval'}
];

/* Визначення батьківського тегу для HTML */
function getCurrentParentTag(cm){
  const cur = cm.getCursor();
  let stack = [];
  for(let i=0;i<=cur.line;i++){
    const line = cm.getLine(i);
    const openTags = [...line.matchAll(/<([a-zA-Z0-9\-:_]+)([^>]*)>/g)];
    openTags.forEach(m=>{
      const tag = m[1].toLowerCase();
      if(!m[0].startsWith('</')) stack.push(tag);
    });
    const closeTags = [...line.matchAll(/<\/([a-zA-Z0-9\-:_]+)>/g)];
    closeTags.forEach(m=>{
      const tag = m[1].toLowerCase();
      const idx = stack.lastIndexOf(tag);
      if(idx!==-1) stack.splice(idx,1);
    });
  }
  return stack.length ? stack[stack.length-1] : 'body';
}

/* Автопідказки та автозакриття */
const VOID_TAGS = new Set(['area','base','br','col','embed','hr','img','input','link','meta','param','source','track','wbr']);
editor.on('inputRead', function(cm, change){
  if(!change || !change.text) return;
  const inserted = change.text.join('');
  if(!inserted.match(/[a-zA-Z<]/)) return;

  const cur = cm.getCursor();
  const token = cm.getTokenAt(cur);

  /* Автозакриття тегів */
  const tag = (function(){
    const line = cm.getLine(cur.line).slice(0, cur.ch);
    const m = line.match(/<([A-Za-z0-9\-:_]+)([^<>]*)>$/);
    if(!m) return null;
    const t = m[1].toLowerCase();
    if(line.endsWith('</'+t+'>')) return null;
    return t;
  })();
  if(tag && !VOID_TAGS.has(tag)){
    const nextChars = cm.getLine(cur.line).slice(cur.ch, cur.ch + tag.length +3);
    if(!nextChars.startsWith('</'+tag)){
      cm.replaceRange(`</${tag}>`, cur);
      cm.setCursor(cur);
    }
  }

  /* Підказки */
  let hintsList = [];
  if(currentLang==='html'){
    const parent = getCurrentParentTag(cm);
    const list = htmlTags[parent] || htmlTags['body'];
    hintsList = list.filter(t=>t.startsWith(token.string.replace('<',''))).map(t=>({text:'<'+t+'>'}));
  }
  if(currentLang==='css') hintsList = cssHints.filter(i=>i.text.startsWith(token.string));
  if(currentLang==='js') hintsList = jsHints.filter(i=>i.text.startsWith(token.string));

  if(hintsList.length){
    CodeMirror.showHint(cm, function(){
      return {list:hintsList, from:CodeMirror.Pos(cur.line, token.start), to:CodeMirror.Pos(cur.line, token.end)};
    }, {completeSingle:false});
  }
});

/* Вставка бази при "!" на місці курсора */
function handleTab(cm){
  const cur = cm.getCursor();
  const line = cm.getLine(cur.line);
  if(line.trim()==='!'){
    const base=`<!DOCTYPE html>
<html lang="uk">
<head>
  <meta charset="UTF-8">
  <title>Document</title>
</head>
<body>

</body>
</html>`;
    cm.replaceRange(base, cur); // вставка на місці курсора
    cm.setCursor({line: cur.line + 7, ch:0}); // курсор після вставки
    return;
  }
  cm.execCommand("defaultTab");
}

/* Live Preview */
function updatePreview(){
  const iframe = document.getElementById('live-preview');
  const doc = iframe.contentDocument || iframe.contentWindow.document;
  doc.open();
  doc.write(`<style>${code.css}</style>${code.html}<script>${code.js}<\/script>`);
  doc.close();
}
</script>
</body>
</html>
