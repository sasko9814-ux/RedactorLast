<!DOCTYPE html>
<html lang="uk">
<head>
<meta charset="UTF-8">
<title>Редактор коду з логотипом</title>
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/codemirror/5.65.14/codemirror.min.css">
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/codemirror/5.65.14/theme/dracula.min.css">
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/codemirror/5.65.14/theme/eclipse.min.css">
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/codemirror/5.65.14/addon/hint/show-hint.min.css">

<style>
* { box-sizing:border-box; margin:0; padding:0; }
body { display:flex; height:100vh; font-family:Segoe UI, sans-serif; background:#1e1e1e; color:#fff; }
#editor-container { width:50%; display:flex; flex-direction:column; border-right:2px solid #333; }
#editor-toolbar { display:flex; gap:5px; align-items:center; padding:5px; background:#2d2d2d; border-bottom:2px solid #333; }
#editor-toolbar button { padding:5px 10px; border:none; border-radius:5px; font-weight:bold; cursor:pointer; transition:0.2s; }
#editor-toolbar button:hover { opacity:0.8; }
#download-btn { background:#28a745; color:#fff; }
#upload-btn { background:#007bff; color:#fff; }
#theme-btn { background:#6f42c1; color:#fff; }
#editor-logo { height:30px; margin-right:10px; user-select:none; }

.tabs { display:flex; border-bottom:2px solid #333; }
.tab { padding:10px 20px; cursor:pointer; font-weight:bold; color:#fff; transition:0.2s; }
.tab:hover { opacity:0.8; }
.tab.active { border-bottom:3px solid #fff; }
.tab[data-lang="html"]{background:#28a745;}
.tab[data-lang="css"]{background:#007bff;}
.tab[data-lang="js"]{background:#fd7e14;}
.CodeMirror { flex:1; height:calc(100vh - 80px); }
#preview { width:50%; background:#f9f9f9; overflow:auto; }
#preview iframe { width:100%; height:100%; border:none; }

/* Світла тема */
.light-theme body { background:#f0f0f0; color:#000; }
.light-theme #editor-container, .light-theme .CodeMirror { background:#fff; color:#000; }
.light-theme #editor-toolbar { background:#ddd; border-bottom-color:#aaa; }
.light-theme .tab { color:#000; }
</style>
</head>
<body>

<div id="editor-container">
  <div id="editor-toolbar">
    <img id="editor-logo" src="https://i.imgur.com/4AiXzf8.png" alt="CodeLab Logo">
    <button id="download-btn">Завантажити файл</button>
    <button id="upload-btn">Вивантажити файл</button>
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
<script src="https://cdnjs.cloudflare.com/ajax/libs/codemirror/5.65.14/mode/htmlmixed/htmlmixed.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/codemirror/5.65.14/mode/javascript/javascript.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/codemirror/5.65.14/mode/css/css.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/codemirror/5.65.14/addon/hint/show-hint.min.js"></script>

<script>
const editor = CodeMirror.fromTextArea(document.getElementById('code-editor'), {
  lineNumbers:true,
  mode:"htmlmixed",
  theme:"dracula",
  extraKeys:{"Ctrl-Space":"autocomplete","Tab":handleTab}
});

const tabs=document.querySelectorAll('.tab');
let currentLang='html';
let code={html:'',css:'',js:''};

// Автозавантаження з LocalStorage
if(localStorage.getItem('codeEditorData')){
  code=JSON.parse(localStorage.getItem('codeEditorData'));
  editor.setValue(code[currentLang]);
  updatePreview();
}

tabs.forEach(tab=>{
  tab.addEventListener('click',()=>{
    tabs.forEach(t=>t.classList.remove('active'));
    tab.classList.add('active');
    currentLang=tab.dataset.lang;
    editor.setOption('mode',currentLang==='html'?'htmlmixed':currentLang);
    editor.setValue(code[currentLang]);
  });
});

editor.on('change',()=>{
  code[currentLang]=editor.getValue();
  localStorage.setItem('codeEditorData',JSON.stringify(code));
  updatePreview();
});

// Завантаження/вивантаження
document.getElementById('download-btn').addEventListener('click',()=>{
  const blob=new Blob([editor.getValue()],{type:'text/plain'});
  const url=URL.createObjectURL(blob);
  const a=document.createElement('a');
  a.href=url;
  a.download=`${currentLang}_code.txt`;
  a.click();
  URL.revokeObjectURL(url);
});

document.getElementById('upload-btn').addEventListener('click',()=>{document.getElementById('file-input').click();});
document.getElementById('file-input').addEventListener('change',e=>{
  const file=e.target.files[0];
  if(!file) return;
  const reader=new FileReader();
  reader.onload=function(ev){editor.setValue(ev.target.result);}
  reader.readAsText(file);
});

// Кнопка тема
document.getElementById('theme-btn').addEventListener('click',()=>{
  document.body.classList.toggle('light-theme');
  editor.setOption('theme', document.body.classList.contains('light-theme') ? 'eclipse' : 'dracula');
});

// Підказки
const hintsData={
  html:['<!DOCTYPE html>','<html>','<head>','<meta>','<title>','<link>','<script>','<style>','<body>','<div>','<span>','<p>','<h1>','<h2>','<h3>','<ul>','<li>','<a href="">','<img src="" alt="">'],
  css:['color','background-color','margin','padding','font-size','font-family','display','position','width','height','border','border-radius','overflow','text-align'],
  js:['function','return','var','let','const','if','else','for','while','console.log()','document.getElementById()','document.querySelector()']
};

// Автозавершення CSS
editor.on('keydown',function(cm,event){
  if(currentLang==='css' && event.key==='Tab'){
    const cur=cm.getCursor();
    const token=cm.getTokenAt(cur);
    const prop=token.string.trim();
    if(hintsData.css.includes(prop)){
      event.preventDefault();
      let value='black';
      if(prop==='background-color') value='white';
      if(prop==='margin'||prop==='padding') value='0px';
      cm.replaceRange(`${prop}: ${value};`,{line:cur.line,ch:token.start},{line:cur.line,ch:token.end});
    }
  }
});

editor.on('inputRead',function(cm,change){
  if(currentLang==='html') autoCloseTags(change);
  if(change.text[0].match(/[a-zA-Z<]/)){
    const cur=editor.getCursor();
    const token=editor.getTokenAt(cur);
    const list=hintsData[currentLang].filter(item=>item.startsWith(token.string));
    if(list.length){
      CodeMirror.showHint(cm,function(){return{list:list,from:CodeMirror.Pos(cur.line,token.start),to:CodeMirror.Pos(cur.line,token.end)}},{completeSingle:false});
    }
  }
});

// Автозакриття тегів
function autoCloseTags(change){
  const line=editor.getLine(editor.getCursor().line);
  const match=line.match(/<([a-zA-Z]+)[^>]*>$/);
  if(match && !line.includes(`</${match[1]}>`)){
    const cur=editor.getCursor();
    editor.replaceRange(`</${match[1]}>`,cur);
  }
}

// Вставка основи документа при "!"
function handleTab(cm){
  const cur=cm.getCursor();
  const line=cm.getLine(cur.line);
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
    cm.replaceRange(base,{line:0,ch:0},{line:cm.lineCount(),ch:0});
    cm.setCursor({line:7,ch:0});
    return;
  }
  cm.execCommand("defaultTab");
}

// Оновлення прев’ю
function updatePreview(){
  const iframe=document.getElementById('live-preview');
  const doc=iframe.contentDocument || iframe.contentWindow.document;
  doc.open();
  doc.write(`<style>${code.css}</style>${code.html}<script>${code.js}<\/script>`);
  doc.close();
}
</script>
</body>
</html>
