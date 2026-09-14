<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>Graph Visualizer</title>
<style>
  * { box-sizing: border-box; }
  :root {
    --bg: #0b1020;
    --panel: #11182b;
    --panel2: #172039;
    --border: #2a3554;
    --text: #e8edf8;
    --muted: #8d9ab8;
    --accent: #7c9cff;
    --danger: #ff6b7a;
    --success: #48d597;
    --node: #6f8cff;
    --edge: #64718f;
    --highlight: #ffd166;
  }
  body {
    margin: 0;
    height: 100vh;
    width: full;
    overflow: hidden;
    font-family: Inter, ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
    background: var(--bg);
    color: var(--text);
  }
  .app { height: 100vh; display: flex; flex-direction: column; }
  header {
    height: 58px; flex: 0 0 58px; display: flex; align-items: center;
    justify-content: space-between; padding: 0 18px;
    border-bottom: 1px solid var(--border); background: #0d1324;
  }
  header h1 { font-size: 17px; margin: 0; letter-spacing: .2px; }
  header .sub { color: var(--muted); font-size: 12px; margin-left: 10px; }
  .header-right { display:flex; gap:8px; }
  .layout { flex: 1; min-height: 0; display:grid; grid-template-columns: 310px 1fr; }
  aside {
    overflow-y:auto; padding: 14px; border-right:1px solid var(--border);
    background: var(--panel);
  }
  .section { margin-bottom: 15px; }
  .section-title {
    text-transform: uppercase; font-size: 10px; letter-spacing: 1px;
    color: var(--muted); margin: 0 0 8px;
  }
  .row { display:flex; gap:7px; margin-bottom:7px; }
  .row > * { flex:1; min-width:0; }
  input, select, textarea, button {
    width:100%; border:1px solid var(--border); border-radius:7px;
    background:var(--panel2); color:var(--text); padding:8px 9px;
    font: inherit; font-size:12px; outline:none;
  }
  textarea { resize:vertical; min-height:110px; line-height:1.45; }
  input:focus, select:focus, textarea:focus { border-color:var(--accent); }
  button { cursor:pointer; transition:.15s; }
  button:hover { filter:brightness(1.14); border-color:#45547b; }
  button.primary { background:#314b9a; border-color:#4b66bd; }
  button.danger { color:#ff9aa5; }
  button.success { color:#76e7b6; }
  .toggle {
    display:flex; align-items:center; justify-content:space-between;
    padding:8px 9px; border:1px solid var(--border); border-radius:7px;
    background:var(--panel2); font-size:12px; margin-bottom:7px;
  }
  .toggle input { width:auto; }
  .hint { color:var(--muted); font-size:11px; line-height:1.45; margin-top:6px; }
  .stats {
    display:grid; grid-template-columns:1fr 1fr; gap:6px;
  }
  .stat { background:var(--panel2); border:1px solid var(--border); border-radius:7px; padding:8px; }
  .stat b { display:block; font-size:15px; }
  .stat span { color:var(--muted); font-size:10px; }
  main { position:relative; min-width:0; min-height:0; }
  svg { width:100%; height:100%; display:block; background:
      radial-gradient(circle at 50% 45%, rgba(45,65,120,.12), transparent 45%),
      #090e1b;
    cursor:default;
  }
  .toolbar {
    position:absolute; left:12px; top:12px; display:flex; gap:6px; z-index:5;
  }
  .toolbar button { width:auto; padding:7px 9px; background:rgba(17,24,43,.9); backdrop-filter:blur(8px); }
  .legend {
    position:absolute; right:12px; bottom:12px; z-index:5;
    background:rgba(17,24,43,.9); border:1px solid var(--border);
    border-radius:8px; padding:8px 10px; font-size:10px; color:var(--muted);
  }
  .legend div { display:flex; align-items:center; gap:7px; margin:3px 0; }
  .dot { width:8px; height:8px; border-radius:50%; display:inline-block; }
  .dot.node { background:var(--node); }
  .dot.selected { background:var(--highlight); }
  .dot.path { background:var(--success); }
  .edge { stroke:var(--edge); stroke-width:2; opacity:.8; }
  .edge.highlight { stroke:var(--highlight); stroke-width:4; opacity:1; }
  .edge.path { stroke:var(--success); stroke-width:4; opacity:1; }
  .node circle { fill:var(--node); stroke:#b8c5ff; stroke-width:1.5; }
  .node text { fill:#fff; font-size:12px; font-weight:700; text-anchor:middle; dominant-baseline:middle; pointer-events:none; }
  .node.selected circle { fill:var(--highlight); stroke:#fff; }
  .node.start circle { fill:var(--success); }
  .node.dragging circle { stroke:#fff; stroke-width:3; }
  .weight {
    fill:#ffffff; font-size:12px; font-weight:700; text-anchor:middle;
    dominant-baseline:middle; paint-order:stroke; stroke:#090e1b;
    stroke-width:5px; stroke-linejoin:round; pointer-events:none;
  }
  .empty {
    position:absolute; inset:0; display:flex; align-items:center; justify-content:center;
    pointer-events:none; color:var(--muted); text-align:center;
  }
  .empty div { max-width:390px; }
  kbd { padding:1px 5px; border:1px solid var(--border); border-radius:4px; background:var(--panel2); }
  @media(max-width:850px) {
    .layout { grid-template-columns: 250px 1fr; }
  }
</style>
</head>
<body>
<div class="app">
<header>
  <div><h1>Graph Visualizer <span class="sub">generic DSA playground</span></h1></div>
  <div class="header-right">
    <button id="exportBtn">Export JSON</button>
    <button id="clearBtn" class="danger">Clear</button>
  </div>
</header>

<div class="layout">
<aside>
  <div class="section">
    <div class="section-title">Graph settings</div>
    <div class="toggle"><span>Directed</span><input id="directed" type="checkbox" checked></div>
    <div class="toggle"><span>Weighted</span><input id="weighted" type="checkbox"></div>
    <div class="toggle"><span>Allow self-loops</span><input id="loops" type="checkbox"></div>
  </div>

  <div class="section">
    <div class="section-title">Add node</div>
    <div class="row">
      <input id="nodeId" placeholder="Label e.g. 1 / A" />
      <button id="addNode" class="primary">Add</button>
    </div>
    <div class="hint">Or double-click an empty area in the canvas.</div>
  </div>

  <div class="section">
    <div class="section-title">Add edge</div>
    <div class="row">
      <input id="from" placeholder="From" />
      <input id="to" placeholder="To" />
    </div>
    <div class="row">
      <input id="weight" placeholder="Weight (optional)" type="number" value="1" />
      <button id="addEdge" class="primary">Connect</button>
    </div>
  </div>

  <div class="section">
    <div class="section-title">Import edge list</div>
    <textarea id="edgeInput" placeholder="1 2 5
2 3 4
3 1 -2

# comments are ignored
# format: from to [weight]"></textarea>
    <div class="row" style="margin-top:7px">
      <button id="importBtn" class="primary">Build graph</button>
      <button id="exampleBtn">Example</button>
    </div>
    <div class="hint">Works with unweighted, weighted, directed and undirected edge lists.</div>
  </div>

  <div class="section">
    <div class="section-title">Visualization</div>
    <div class="row">
      <button id="randomBtn">Random layout</button>
      <button id="fitBtn">Fit view</button>
    </div>
    <div class="row">
      <button id="clearHighlights">Clear highlights</button>
      <button id="cycleBtn" class="success">Highlight cycle</button>
    </div>
  </div>

  <div class="section">
    <div class="section-title">Stats</div>
    <div class="stats">
      <div class="stat"><b id="nodeCount">0</b><span>Nodes</span></div>
      <div class="stat"><b id="edgeCount">0</b><span>Edges</span></div>
      <div class="stat"><b id="degreeInfo">—</b><span>Max degree</span></div>
      <div class="stat"><b id="modeInfo">D</b><span>Mode</span></div>
    </div>
  </div>

  <div class="hint">
    <b>Mouse:</b> drag nodes · click to select · double-click empty space to add<br>
    <b>Keyboard:</b> <kbd>Delete</kbd> removes selected node · <kbd>Esc</kbd> clears selection
  </div>
</aside>

<main id="canvasWrap">
  <div class="toolbar">
    <button id="zoomIn">＋</button>
    <button id="zoomOut">－</button>
    <button id="resetZoom">Reset</button>
  </div>
  <svg id="svg" xmlns="http://www.w3.org/2000/svg">
    <defs>
      <marker id="arrow" markerWidth="9" markerHeight="7" refX="8" refY="3.5" orient="auto">
        <polygon points="0 0, 9 3.5, 0 7" fill="#64718f"></polygon>
      </marker>
      <marker id="arrowHighlight" markerWidth="9" markerHeight="7" refX="8" refY="3.5" orient="auto">
        <polygon points="0 0, 9 3.5, 0 7" fill="#ffd166"></polygon>
      </marker>
      <marker id="arrowPath" markerWidth="9" markerHeight="7" refX="8" refY="3.5" orient="auto">
        <polygon points="0 0, 9 3.5, 0 7" fill="#48d597"></polygon>
      </marker>
    </defs>
    <g id="world">
      <g id="edges"></g>
      <g id="nodes"></g>
    </g>
  </svg>
  <div class="empty" id="emptyState">
    <div>
      <b>Add a graph to get started</b><br>
      Import an edge list, add nodes manually, or double-click the canvas.
    </div>
  </div>
  <div class="legend">
    <div><i class="dot node"></i> Node</div>
    <div><i class="dot selected"></i> Selected</div>
    <div><i class="dot path"></i> Cycle / highlighted path</div>
  </div>
</main>
</div>
</div>

<script>
const svg = document.getElementById('svg');
const world = document.getElementById('world');
const edgeLayer = document.getElementById('edges');
const nodeLayer = document.getElementById('nodes');

let nodes = new Map();
let edges = [];
let selected = new Set();
let highlightedEdges = new Set();
let pathEdges = new Set();
let scale = 1, tx = 0, ty = 0;
let drag = null;

const $ = id => document.getElementById(id);

function resize() {
  render();
}
window.addEventListener('resize', resize);

function svgPoint(e) {
  const r = svg.getBoundingClientRect();
  return { x:(e.clientX-r.left-tx)/scale, y:(e.clientY-r.top-ty)/scale };
}

function makeId(x) {
  return String(x).trim();
}

function addNode(id, x, y) {
  id = makeId(id);
  if (!id || nodes.has(id)) return false;
  const r = svg.getBoundingClientRect();
  nodes.set(id, {
    id,
    x: x ?? ((r.width/2 - tx)/scale + (Math.random()-.5)*100),
    y: y ?? ((r.height/2 - ty)/scale + (Math.random()-.5)*100)
  });
  update();
  return true;
}

function addEdge(a,b,w=1) {
  a=makeId(a); b=makeId(b);
  if (!a || !b || !nodes.has(a) || !nodes.has(b)) return false;
  if (!$('loops').checked && a===b) return false;
  if (!$('weighted').checked) w=1;
  edges.push({a,b,w:Number(w)||0});
  update();
  return true;
}

function removeSelected() {
  if (!selected.size) return;
  for (const id of selected) nodes.delete(id);
  edges = edges.filter(e => nodes.has(e.a) && nodes.has(e.b));
  selected.clear();
  highlightedEdges.clear();
  pathEdges.clear();
  update();
}

function updateStats() {
  $('nodeCount').textContent = nodes.size;
  $('edgeCount').textContent = edges.length;
  const deg = {};
  for (const n of nodes.keys()) deg[n]=0;
  edges.forEach(e => { if(deg[e.a]!==undefined)deg[e.a]++; if(deg[e.b]!==undefined && e.b!==e.a)deg[e.b]++; });
  $('degreeInfo').textContent = nodes.size ? Math.max(...Object.values(deg)) : '—';
  $('modeInfo').textContent = ($('directed').checked?'Directed':'Undirected') + ($('weighted').checked?' · W':'');
  $('emptyState').style.display = nodes.size ? 'none' : 'flex';
}

function edgeKey(e) {
  return e._id;
}

function render() {
  const dir = $('directed').checked;
  world.setAttribute('transform', `translate(${tx} ${ty}) scale(${scale})`);
  edgeLayer.innerHTML='';
  nodeLayer.innerHTML='';

  edges.forEach((e,i)=>{
    e._id = i;
    const A=nodes.get(e.a), B=nodes.get(e.b);
    if(!A||!B)return;
    const line=document.createElementNS('http://www.w3.org/2000/svg','line');
    let x1=A.x,y1=A.y,x2=B.x,y2=B.y;

    const dx=x2-x1,dy=y2-y1, len=Math.hypot(dx,dy)||1;
    const ux=dx/len,uy=dy/len;
    const rr=21;
    x1+=ux*rr;y1+=uy*rr;x2-=ux*rr;y2-=uy*rr;

    line.setAttribute('x1',x1);line.setAttribute('y1',y1);
    line.setAttribute('x2',x2);line.setAttribute('y2',y2);
    line.classList.add('edge');
    if(highlightedEdges.has(i))line.classList.add('highlight');
    if(pathEdges.has(i))line.classList.add('path');

    if(dir){
      line.setAttribute('marker-end', pathEdges.has(i) ? 'url(#arrowPath)' :
        highlightedEdges.has(i) ? 'url(#arrowHighlight)' : 'url(#arrow)');
    }

    if(A===B){
      line.setAttribute('x1',A.x+18);line.setAttribute('y1',A.y-18);
      line.setAttribute('x2',A.x+18);line.setAttribute('y2',A.y+2);
    }

    edgeLayer.appendChild(line);

    if($('weighted').checked){
      const t=document.createElementNS('http://www.w3.org/2000/svg','text');
      t.classList.add('weight');
      t.setAttribute('x',(A.x+B.x)/2);
      t.setAttribute('y',(A.y+B.y)/2-7);
      t.textContent=e.w;
      edgeLayer.appendChild(t);
    }
  });

  for(const [id,n] of nodes){
    const g=document.createElementNS('http://www.w3.org/2000/svg','g');
    g.classList.add('node');
    if(selected.has(id))g.classList.add('selected');
    g.dataset.id=id;
    const c=document.createElementNS('http://www.w3.org/2000/svg','circle');
    c.setAttribute('cx',n.x);c.setAttribute('cy',n.y);c.setAttribute('r',19);
    const t=document.createElementNS('http://www.w3.org/2000/svg','text');
    t.setAttribute('x',n.x);t.setAttribute('y',n.y);t.textContent=id;
    g.append(c,t);
    nodeLayer.appendChild(g);
  }
  updateStats();
}

function update(){ render(); }

function clearAll(){
  nodes.clear(); edges=[]; selected.clear(); highlightedEdges.clear(); pathEdges.clear();
  tx=0;ty=0;scale=1;update();
}

$('addNode').onclick=()=>{
  const id=$('nodeId').value.trim();
  if(addNode(id)) $('nodeId').value='';
  else alert('Node label is empty or already exists.');
};
$('nodeId').addEventListener('keydown',e=>{if(e.key==='Enter')$('addNode').click()});

$('addEdge').onclick=()=>{
  const ok=addEdge($('from').value,$('to').value,$('weight').value);
  if(!ok) alert('Both nodes must exist. Check the labels.');
};

$('importBtn').onclick=()=>{
  const lines=$('edgeInput').value.split(/\n/);
  clearAll();
  const parsed=[];
  for(const raw of lines){
    const line=raw.trim();
    if(!line || line.startsWith('#'))continue;
    const p=line.split(/\s+/);
    if(p.length>=2) parsed.push(p);
  }
  // If any imported edge has a third column, treat the graph as weighted.
  if (parsed.some(p => p.length >= 3)) {
    $('weighted').checked = true;
  }

  const ids=[...new Set(parsed.flatMap(p=>[p[0],p[1]]))];
  const r=svg.getBoundingClientRect();
  const cx=r.width/2,cy=r.height/2;
  ids.forEach((id,i)=>{
    const angle=i/Math.max(ids.length,1)*Math.PI*2;
    addNode(id,(cx-0)/scale+Math.cos(angle)*Math.min(220,120+ids.length*8),
      (cy-0)/scale+Math.sin(angle)*Math.min(220,120+ids.length*8));
  });
  for(const p of parsed)addEdge(p[0],p[1],p[2]??1);
  fitView();
};

$('exampleBtn').onclick=()=>{
  $('directed').checked=true;$('weighted').checked=true;
  $('edgeInput').value=`1 2 1
2 4 1
3 1 1
4 1 -3
4 3 -2`;
};

$('clearBtn').onclick=clearAll;
$('clearHighlights').onclick=()=>{highlightedEdges.clear();pathEdges.clear();update()};
$('randomBtn').onclick=()=>{
  const r=svg.getBoundingClientRect();
  for(const n of nodes.values()){
    n.x=70+Math.random()*Math.max(100,r.width/scale-140);
    n.y=70+Math.random()*Math.max(100,r.height/scale-140);
  }
  update();
};
$('fitBtn').onclick=fitView;

$('directed').onchange=update;
$('weighted').onchange=update;

function fitView(){
  if(!nodes.size){tx=0;ty=0;scale=1;render();return;}
  const r=svg.getBoundingClientRect();
  const arr=[...nodes.values()];
  const minX=Math.min(...arr.map(n=>n.x)),maxX=Math.max(...arr.map(n=>n.x));
  const minY=Math.min(...arr.map(n=>n.y)),maxY=Math.max(...arr.map(n=>n.y));
  const w=Math.max(maxX-minX,100),h=Math.max(maxY-minY,100);
  scale=Math.min(.88*r.width/w,.88*r.height/h,2);
  tx=r.width/2-scale*(minX+maxX)/2;
  ty=r.height/2-scale*(minY+maxY)/2;
  render();
}

function highlightCycle(){
  highlightedEdges.clear();pathEdges.clear();
  const adj=new Map();
  for(const id of nodes.keys())adj.set(id,[]);
  edges.forEach((e,i)=>{adj.get(e.a).push([e.b,i]); if(!$('directed').checked)adj.get(e.b).push([e.a,i]);});
  const state=new Map(), parent=new Map();
  let found=null;

  function dfs(u){
    state.set(u,1);
    for(const [v,ei] of adj.get(u)||[]){
      if(found)return;
      if(state.get(v)===1){
        const ids=[ei];
        let cur=u;
        while(cur!==v){
          const p=parent.get(cur);
          if(!p)break;
          ids.push(p.ei);cur=p.u;
        }
        found=ids;
        return;
      }
      if(!state.get(v)){
        parent.set(v,{u,ei});
        dfs(v);
      }
    }
    state.set(u,2);
  }
  for(const id of nodes.keys()) if(!state.get(id)) dfs(id);
  if(found) found.forEach(i=>pathEdges.add(i));
  else alert('No cycle found in the current graph.');
  render();
}
$('cycleBtn').onclick=highlightCycle;

svg.addEventListener('dblclick',e=>{
  if(e.target.closest('.node'))return;
  const p=svgPoint(e);
  const id=prompt('Node label:');
  if(id)addNode(id,p.x,p.y);
});

svg.addEventListener('click',e=>{
  const g=e.target.closest('.node');
  if(!g){
    if(!e.shiftKey)selected.clear();
    render(); return;
  }
  const id=g.dataset.id;
  if(e.shiftKey){
    if(selected.has(id))selected.delete(id);else selected.add(id);
  }else{
    selected.clear();selected.add(id);
  }
  render();
});

svg.addEventListener('mousedown',e=>{
  const g=e.target.closest('.node');
  if(!g)return;
  const id=g.dataset.id,p=svgPoint(e),n=nodes.get(id);
  drag={id,dx:n.x-p.x,dy:n.y-p.y};
  g.classList.add('dragging');
  e.preventDefault();
});
window.addEventListener('mousemove',e=>{
  if(!drag)return;
  const p=svgPoint(e),n=nodes.get(drag.id);
  n.x=p.x+drag.dx;n.y=p.y+drag.dy;
  render();
});
window.addEventListener('mouseup',()=>{
  if(drag){const n=[...nodeLayer.querySelectorAll('.node')].find(x=>x.dataset.id===drag.id);if(n)n.classList.remove('dragging');}
  drag=null;
});

svg.addEventListener('wheel',e=>{
  e.preventDefault();
  const p=svgPoint(e);
  const factor=e.deltaY<0?1.12:.89;
  const ns=Math.max(.25,Math.min(3,scale*factor));
  tx=e.clientX-svg.getBoundingClientRect().left-p.x*ns;
  ty=e.clientY-svg.getBoundingClientRect().top-p.y*ns;
  scale=ns;render();
},{passive:false});

$('zoomIn').onclick=()=>{scale=Math.min(3,scale*1.15);render()};
$('zoomOut').onclick=()=>{scale=Math.max(.25,scale*.87);render()};
$('resetZoom').onclick=()=>{scale=1;tx=0;ty=0;render()};

window.addEventListener('keydown',e=>{
  if(e.key==='Delete' || e.key==='Backspace') removeSelected();
  if(e.key==='Escape'){selected.clear();highlightedEdges.clear();pathEdges.clear();render();}
});

$('exportBtn').onclick=()=>{
  const data={
    directed:$('directed').checked,
    weighted:$('weighted').checked,
    nodes:[...nodes.values()],
    edges:edges.map(({a,b,w})=>({a,b,w}))
  };
  const blob=new Blob([JSON.stringify(data,null,2)],{type:'application/json'});
  const a=document.createElement('a');a.href=URL.createObjectURL(blob);a.download='graph.json';a.click();
  URL.revokeObjectURL(a.href);
};

update();
</script>
</body>
</html>
