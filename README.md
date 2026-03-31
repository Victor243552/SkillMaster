# SkillMaster
这是一个本地技能记录工具，不用安装、数据安全、打开即用 能记录技能名称、学习阶段、学习进度 帮你看见自己的学习积累，对抗三分钟热度

<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
  <title>技能管理系统</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <link href="https://cdn.jsdelivr.net/npm/font-awesome@4.7.0/css/font-awesome.min.css" rel="stylesheet">
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body { background: #F9FAFC; font-family: system-ui, sans-serif; }
    .card-shadow { box-shadow: 0 4px 15px rgba(0,0,0,0.05); }
    .circle-progress { transform: rotate(-90deg); }
  </style>
</head>

<!-- 登录注册 -->
<div id="authPage" class="min-h-screen flex items-center justify-center p-4">
  <div class="w-full max-w-sm bg-white rounded-2xl p-6 card-shadow">
    <h2 class="text-center text-xl font-bold text-gray-800">技能管理系统</h2>
    <p class="text-center text-gray-500 text-sm mb-6">20小时养成计划</p>
    
    <div id="loginForm">
      <input id="loginUser" placeholder="用户名" class="w-full p-3 rounded-lg border mb-3">
      <input id="loginPwd" type="password" placeholder="密码" class="w-full p-3 rounded-lg border mb-4">
      <button onclick="userLogin()" class="w-full bg-blue-600 text-white p-3 rounded-lg">登录</button>
      <p class="text-xs text-center mt-3 text-gray-500">没有账号？<span onclick="switchToRegister()" class="text-blue-600 cursor-pointer">立即注册</span></p>
    </div>

    <div id="registerForm" class="hidden">
      <input id="regUser" placeholder="设置用户名" class="w-full p-3 rounded-lg border mb-3">
      <input id="regPwd" type="password" placeholder="设置密码" class="w-full p-3 rounded-lg border mb-4">
      <button onclick="userRegister()" class="w-full bg-blue-600 text-white p-3 rounded-lg">注册</button>
      <p class="text-xs text-center mt-3 text-gray-500">已有账号？<span onclick="switchToLogin()" class="text-blue-600 cursor-pointer">返回登录</span></p>
    </div>
  </div>
</div>

<!-- 主界面 -->
<div id="mainPage" class="hidden pb-20">
  <div class="bg-white p-4 flex justify-between items-center border-b">
    <div>
      <h1 class="font-bold text-lg">我的技能</h1>
      <p class="text-sm text-blue-600" id="currentUserName"></p>
    </div>
    <button onclick="userLogout()" class="px-3 py-2 border rounded-lg text-sm">退出登录</button>
  </div>

  <div class="p-4 grid grid-cols-3 gap-2">
    <div class="bg-white p-3 rounded-xl card-shadow text-center">
      <p class="text-xs text-gray-500">今日学习</p>
      <p id="todayTime" class="font-bold mt-1">0h 0m</p>
    </div>
    <div class="bg-white p-3 rounded-xl card-shadow text-center">
      <p class="text-xs text-gray-500">本周学习</p>
      <p id="weekTime" class="font-bold mt-1">0h 0m</p>
    </div>
    <div class="bg-white p-3 rounded-xl card-shadow text-center">
      <p class="text-xs text-gray-500">本月学习</p>
      <p id="monthTime" class="font-bold mt-1">0h 0m</p>
    </div>
  </div>

  <div class="px-4 flex gap-2 mb-4 overflow-x-auto">
    <button class="cate-btn active px-3 py-2 bg-blue-600 text-white rounded-lg" data-type="all">全部</button>
    <button class="cate-btn px-3 py-2 bg-gray-100 rounded-lg" data-type="学习">学习</button>
    <button class="cate-btn px-3 py-2 bg-gray-100 rounded-lg" data-type="工作">工作</button>
    <button class="cate-btn px-3 py-2 bg-gray-100 rounded-lg" data-type="爱好">爱好</button>
    <button onclick="openAddModal()" class="ml-auto px-3 py-2 bg-blue-600 text-white rounded-lg">+ 添加</button>
  </div>

  <div class="px-4 mb-4">
    <div class="bg-white p-4 rounded-xl card-shadow">
      <p class="text-sm font-medium">今日推荐学习</p>
      <p id="todayTask" class="text-xs text-gray-500 mt-1">开始学习吧</p>
    </div>
  </div>

  <div class="px-4 grid grid-cols-1 sm:grid-cols-2 gap-3" id="skillList"></div>
</div>

<!-- 弹窗 -->
<div id="modal" class="fixed inset-0 bg-black/20 hidden items-center justify-center p-4 z-50">
  <div class="bg-white w-full max-w-sm rounded-2xl p-5">
    <h3 class="font-bold mb-3" id="modalTitle">添加项目</h3>
    <input id="modalInput" class="w-full p-3 border rounded-lg mb-3">
    <select id="modalCate" class="w-full p-3 border rounded-lg mb-4">
      <option value="学习">学习</option>
      <option value="工作">工作</option>
      <option value="爱好">爱好</option>
    </select>
    <div class="flex gap-2">
      <button onclick="closeModal()" class="flex-1 p-3 bg-gray-100 rounded-lg">取消</button>
      <button onclick="saveModal()" class="flex-1 p-3 bg-blue-600 text-white rounded-lg">保存</button>
    </div>
  </div>
</div>

<script>
const TARGET_MINUTES = 20 * 60;
let currentUser = '';
let editIndex = -1;

// 切换登录注册
function switchToRegister() {
  document.getElementById('loginForm').classList.add('hidden');
  document.getElementById('registerForm').classList.remove('hidden');
}
function switchToLogin() {
  document.getElementById('registerForm').classList.add('hidden');
  document.getElementById('loginForm').classList.remove('hidden');
}

// 注册
function userRegister() {
  const user = document.getElementById('regUser').value.trim();
  const pwd = document.getElementById('regPwd').value.trim();
  if (!user || !pwd) return alert('请输入用户名和密码');

  let users = JSON.parse(localStorage.getItem('skill_users')) || {};
  if (users[user]) return alert('用户名已存在');

  users[user] = pwd;
  localStorage.setItem('skill_users', JSON.stringify(users));
  alert('注册成功！请登录');
  switchToLogin();
}

// 登录
function userLogin() {
  const user = document.getElementById('loginUser').value.trim();
  const pwd = document.getElementById('loginPwd').value.trim();

  let users = JSON.parse(localStorage.getItem('skill_users')) || {维克多: "123456"};
  if (!users[user] || users[user] !== pwd) return alert('用户名或密码错误');

  currentUser = user;
  document.getElementById('currentUserName').innerText = '欢迎：' + currentUser;
  document.getElementById('authPage').classList.add('hidden');
  document.getElementById('mainPage').classList.remove('hidden');
  initUser();
}

// 退出
function userLogout() {
  currentUser = '';
  document.getElementById('mainPage').classList.add('hidden');
  document.getElementById('authPage').classList.remove('hidden');
}

// 数据管理
function getUserSkills() {
  const data = localStorage.getItem(`skill_data_${currentUser}`);
  return data ? JSON.parse(data) : [
    {name:"复盘焦煤",cate:"工作",m:0},{name:"复盘碳酸锂",cate:"工作",m:0},
    {name:"复盘黄金",cate:"工作",m:0},{name:"复盘白银",cate:"工作",m:0},
    {name:"数学微积分",cate:"学习",m:0},{name:"数学概率学",cate:"学习",m:0},
    {name:"AI课程",cate:"学习",m:0},{name:"英语",cate:"学习",m:0},
    {name:"日语",cate:"学习",m:0},{name:"韩语",cate:"学习",m:0},{name:"游泳",cate:"爱好",m:0}
  ];
}
function saveUserSkills(arr) { localStorage.setItem(`skill_data_${currentUser}`, JSON.stringify(arr)); }
function getUserLogs() { return JSON.parse(localStorage.getItem(`skill_logs_${currentUser}`)) || []; }
function saveUserLogs(arr) { localStorage.setItem(`skill_logs_${currentUser}`, JSON.stringify(arr)); }

// 初始化
function initUser() { renderSkills(); calcStats(); recommendTask(); }

// 渲染列表
function renderSkills() {
  const list = getUserSkills();
  const el = document.getElementById('skillList');
  if (list.length === 0) { el.innerHTML = "<p class='text-center text-gray-400 py-10'>暂无项目</p>"; return; }

  el.innerHTML = list.map((item, i) => {
    const pct = Math.min(100, (item.m / TARGET_MINUTES * 100).toFixed(1));
    const r = 30, c = 2 * Math.PI * r, d = (pct / 100) * c;
    const h = Math.floor(item.m / 60), m = item.m % 60;
    const color = item.cate == "工作" ? "#FF7D00" : item.cate == "爱好" ? "#00B42A" : "#165DFF";

    return `
    <div class="bg-white rounded-xl p-4 card-shadow" data-cate="${item.cate}">
      <div class="flex justify-between items-start mb-3">
        <span class="text-xs px-2 py-1 rounded-full bg-gray-100" style="color:${color}">${item.cate}</span>
        <button onclick="delSkill(${i})" class="text-gray-400"><i class="fa fa-trash-o"></i></button>
      </div>
      <div class="flex items-center gap-3 mb-3">
        <div class="relative w-16 h-16">
          <svg class="circle-progress w-full h-full" viewBox="0 0 70 70">
            <circle cx="35" cy="35" r="${r}" stroke="#eee" stroke-width="5" fill="none"></circle>
            <circle cx="35" cy="35" r="${r}" stroke="${color}" stroke-width="5" fill="none" stroke-dasharray="${c}" stroke-dashoffset="${c-d}"></circle>
          </svg>
          <div class="absolute inset-0 flex items-center justify-center text-sm font-bold">${pct}%</div>
        </div>
        <div>
          <p class="font-bold">${item.name}</p>
          <p class="text-xs text-gray-500">${h}h ${m}m / 20h</p>
        </div>
      </div>
      <div class="w-full h-1.5 bg-gray-100 rounded-full mb-3">
        <div style="width:${pct}%;background:${color}" class="h-full rounded-full"></div>
      </div>
      <button onclick="logSkill(${i})" class="w-full p-2 rounded-lg text-blue-600 bg-blue-50">记录学习</button>
    </div>`;
  }).join('');
}

// 统计
function calcStats() {
  const logs = getUserLogs();
  const now = new Date();
  const today = new Date(now.getFullYear(), now.getMonth(), now.getDate()).getTime();
  const week = new Date(now.setDate(now.getDate() - now.getDay())).getTime();
  const month = new Date(new Date().getFullYear(), new Date().getMonth(), 1).getTime();

  let t = 0, w = 0, m = 0;
  logs.forEach(l => {
    if (l.time >= today) t += l.m;
    if (l.time >= week) w += l.m;
    if (l.time >= month) m += l.m;
  });

  document.getElementById("todayTime").innerText = Math.floor(t/60)+"h "+t%60+"m";
  document.getElementById("weekTime").innerText = Math.floor(w/60)+"h "+w%60+"m";
  document.getElementById("monthTime").innerText = Math.floor(m/60)+"h "+m%60+"m";
}

// 推荐任务
function recommendTask() {
  const list = getUserSkills();
  if (list.length === 0) return;
  const target = [...list].sort((a, b) => a.m/TARGET_MINUTES - b.m/TARGET_MINUTES)[0];
  document.getElementById("todayTask").innerText = `${target.name} | 进度${((target.m/TARGET_MINUTES*100).toFixed(1))}%`;
}

// 弹窗
function openAddModal() { editIndex = -1; document.getElementById('modalTitle').innerText = '添加项目'; document.getElementById('modalInput').value = ''; document.getElementById('modal').style.display = 'flex'; }
function logSkill(i) { editIndex = i; document.getElementById('modalTitle').innerText = '记录学习分钟'; document.getElementById('modalInput').value = ''; document.getElementById('modalInput').type = 'number'; document.getElementById('modalCate').classList.add('hidden'); document.getElementById('modal').style.display = 'flex'; }
function closeModal() { document.getElementById('modal').style.display = 'none'; document.getElementById('modalInput').type = 'text'; document.getElementById('modalCate').classList.remove('hidden'); }

function saveModal() {
  const val = document.getElementById('modalInput').value;
  if (!val) return;
  const list = getUserSkills();
  if (editIndex === -1) {
    list.push({ name: val, cate: document.getElementById('modalCate').value, m: 0 });
  } else {
    list[editIndex].m += Number(val);
    const logs = getUserLogs();
    logs.push({ time: new Date().getTime(), m: Number(val) });
    saveUserLogs(logs);
  }
  saveUserSkills(list);
  closeModal();
  renderSkills(); calcStats(); recommendTask();
}

// 删除
function delSkill(i) {
  if (!confirm('确定删除？')) return;
  const list = getUserSkills();
  list.splice(i, 1);
  saveUserSkills(list);
  renderSkills();
  recommendTask();
}

// 分类筛选
document.querySelectorAll('.cate-btn').forEach(btn => {
  btn.onclick = () => {
    document.querySelectorAll('.cate-btn').forEach(b => b.classList.remove('active', 'bg-blue-600', 'text-white'));
    btn.classList.add('active', 'bg-blue-600', 'text-white');
    const type = btn.dataset.type;
    document.querySelectorAll('[data-cate]').forEach(c => {
      c.style.display = type === 'all' || c.dataset.cate === type ? 'block' : 'none';
    });
  };
});
</script>
</html>