/* =========================================================
   TaskFlow — script.js
   Vanilla JS: CRUD, search, filter, sort, localStorage, modals,
   toasts, loading, empty states, validation, dashboard stats.
   ========================================================= */
(function () {
  'use strict';

  // ─── Constants ───────────────────────────────────────────
  var STORAGE_KEY = 'taskflow.tasks.v1';
  var PRIORITY_RANK = { high: 3, medium: 2, low: 1 };
  var PRIORITY_LABELS = { high: 'High', medium: 'Medium', low: 'Low' };

  // ─── Demo Data ───────────────────────────────────────────
  var demoTasks = [
    { id: 'demo-1',  title: 'Design landing page hero',     desc: 'Create a bold hero with CTA and product preview.',       deadline: addDays(0),  priority: 'high',   category: 'Work',     completed: false, created: Date.now() - 9000 },
    { id: 'demo-2',  title: 'Review team pull requests',     desc: 'Code review for the auth-service and billing repos.',    deadline: addDays(1),  priority: 'medium', category: 'Work',     completed: false, created: Date.now() - 8000 },
    { id: 'demo-3',  title: 'Prepare Q4 presentation',       desc: 'Slides summarizing quarterly metrics and goals.',        deadline: addDays(3),  priority: 'high',   category: 'Work',     completed: false, created: Date.now() - 7000 },
    { id: 'demo-4',  title: 'Read chapter 7 — Algorithms',   desc: 'Graph traversal: BFS, DFS, and shortest-path.',           deadline: addDays(2),  priority: 'medium', category: 'Study',    completed: false, created: Date.now() - 6000 },
    { id: 'demo-5',  title: 'Morning run — 5km',             desc: 'Track pace and heart rate.',                              deadline: addDays(0),  priority: 'low',    category: 'Health',   completed: true,  created: Date.now() - 5000 },
    { id: 'demo-6',  title: 'Buy groceries for the week',     desc: 'Vegetables, oats, eggs, coffee beans.',                   deadline: addDays(-1), priority: 'low',    category: 'Errands',  completed: false, created: Date.now() - 4000 },
    { id: 'demo-7',  title: 'Submit hackathon project',       desc: 'Push source to GitHub and submit the live link.',         deadline: addDays(0),  priority: 'high',   category: 'Work',     completed: false, created: Date.now() - 3000 },
    { id: 'demo-8',  title: 'Call dentist for appointment',   desc: 'Schedule a routine cleaning.',                            deadline: addDays(5),  priority: 'medium', category: 'Personal', completed: false, created: Date.now() - 2000 },
    { id: 'demo-9',  title: 'Set up project repository',      desc: 'Initialize git, add README and CI config.',               deadline: addDays(-2), priority: 'high',   category: 'Work',     completed: true,  created: Date.now() - 1000 },
    { id: 'demo-10', title: 'Plan weekend trip',              desc: 'Book hotel and list nearby hiking trails.',               deadline: addDays(7),  priority: 'low',    category: 'Personal', completed: false, created: Date.now() - 500 },
    { id: 'demo-11', title: 'Finish online course module 3',  desc: 'Complete quizzes and submit the final assignment.',       deadline: addDays(4),  priority: 'medium', category: 'Study',    completed: false, created: Date.now() - 200 },
    { id: 'demo-12', title: 'Organize desk and workspace',    desc: 'Declutter, file loose papers, clean monitors.',           deadline: addDays(-3), priority: 'low',    category: 'Personal', completed: true,  created: Date.now() - 100 }
  ];

  // ─── State ───────────────────────────────────────────────
  var tasks = [];
  var currentFilter = 'all';
  var currentSort = 'dueDate';
  var currentPriorityFilter = 'all';
  var searchQuery = '';
  var pendingConfirm = null;

  // ─── DOM refs ────────────────────────────────────────────
  var $ = function (sel) { return document.querySelector(sel); };
  var taskList, emptyState, toastContainer, loadingScreen, navbar, navLinks, hamburger;

  // ─── Utility ─────────────────────────────────────────────
  function addDays(n) {
    var d = new Date();
    d.setHours(23, 59, 0, 0);
    d.setDate(d.getDate() + n);
    return d.toISOString().split('T')[0];
  }

  function isOverdue(task) {
    if (task.completed) return false;
    var today = new Date(); today.setHours(0, 0, 0, 0);
    var due = new Date(task.deadline + 'T00:00:00');
    return due < today;
  }

  function daysUntil(deadline) {
    var today = new Date(); today.setHours(0, 0, 0, 0);
    var due = new Date(deadline + 'T00:00:00');
    return Math.round((due - today) / 86400000);
  }

  function deadlineLabel(deadline) {
    var d = daysUntil(deadline);
    if (d < 0) return Math.abs(d) + 'd overdue';
    if (d === 0) return 'Due today';
    if (d === 1) return 'Due tomorrow';
    if (d <= 7) return 'Due in ' + d + ' days';
    return 'Due ' + formatDate(deadline);
  }

  function formatDate(s) {
    var d = new Date(s + 'T00:00:00');
    return d.toLocaleDateString('en-US', { month: 'short', day: 'numeric' });
  }

  function uid() { return 't-' + Date.now() + '-' + Math.random().toString(36).slice(2, 7); }

  // ─── localStorage ────────────────────────────────────────
  function loadTasks() {
    try {
      var raw = localStorage.getItem(STORAGE_KEY);
      if (raw) {
        var parsed = JSON.parse(raw);
        if (Array.isArray(parsed)) return parsed;
      }
    } catch (e) { console.warn('Load failed, using demo data', e); }
    return demoTasks.slice();
  }

  function saveTasks() {
    try { localStorage.setItem(STORAGE_KEY, JSON.stringify(tasks)); }
    catch (e) { console.warn('Save failed', e); }
  }

  // ─── Rendering ───────────────────────────────────────────
  function getFilteredTasks() {
    var list = tasks.filter(function (t) {
      // status filter
      if (currentFilter === 'pending' && t.completed) return false;
      if (currentFilter === 'completed' && !t.completed) return false;
      if (currentFilter === 'overdue' && !isOverdue(t)) return false;
      // priority filter
      if (currentPriorityFilter !== 'all' && t.priority !== currentPriorityFilter) return false;
      // search
      if (searchQuery) {
        var q = searchQuery.toLowerCase();
        if (t.title.toLowerCase().indexOf(q) === -1 && t.category.toLowerCase().indexOf(q) === -1 && (t.desc || '').toLowerCase().indexOf(q) === -1) return false;
      }
      return true;
    });

    list.sort(function (a, b) {
      if (currentSort === 'dueDate') return new Date(a.deadline) - new Date(b.deadline);
      if (currentSort === 'priority') return PRIORITY_RANK[b.priority] - PRIORITY_RANK[a.priority];
      if (currentSort === 'created') return b.created - a.created;
      if (currentSort === 'title') return a.title.localeCompare(b.title);
      return 0;
    });
    return list;
  }

  function renderTasks() {
    var list = getFilteredTasks();
    taskList.innerHTML = '';

    if (list.length === 0) {
      taskList.style.display = 'none';
      emptyState.hidden = false;
    } else {
      taskList.style.display = '';
      emptyState.hidden = true;
      list.forEach(function (t) { taskList.appendChild(createCard(t)); });
    }
  }

  function createCard(task) {
    var card = document.createElement('article');
    card.className = 'task-card' + (task.completed ? ' completed' : '');
    card.dataset.priority = task.priority;

    var overdue = isOverdue(task);

    // checkbox
    var check = document.createElement('button');
    check.className = 'task-check' + (task.completed ? ' checked' : '');
    check.setAttribute('role', 'checkbox');
    check.setAttribute('aria-checked', task.completed ? 'true' : 'false');
    check.setAttribute('aria-label', task.completed ? 'Mark as not done' : 'Mark as done');
    check.addEventListener('click', function () { toggleComplete(task.id); });

    // title
    var titleWrap = document.createElement('div');
    titleWrap.className = 'task-card-top';
    var titleSpan = document.createElement('span');
    titleSpan.className = 'task-card-title';
    titleSpan.textContent = task.title;
    titleWrap.appendChild(check);
    titleWrap.appendChild(titleSpan);

    // actions
    var actions = document.createElement('div');
    actions.className = 'task-actions';
    var editBtn = iconBtn('✎', 'Edit task', function () { openModal(task); });
    var delBtn = iconBtn('🗑', 'Delete task', 'danger', function () { confirmDelete(task); });
    actions.appendChild(editBtn);
    actions.appendChild(delBtn);
    titleWrap.appendChild(actions);

    card.appendChild(titleWrap);

    // description
    if (task.desc) {
      var desc = document.createElement('p');
      desc.className = 'task-card-desc';
      desc.textContent = task.desc;
      card.appendChild(desc);
    }

    // badges
    var badges = document.createElement('div');
    badges.className = 'task-badges';
    badges.appendChild(makeBadge('badge-priority-' + task.priority, PRIORITY_LABELS[task.priority]));
    badges.appendChild(makeBadge('badge-category', task.category));
    if (overdue) {
      badges.appendChild(makeBadge('badge-overdue', '⚠ Overdue'));
    } else if (task.completed) {
      badges.appendChild(makeBadge('badge-done', '✓ Done'));
    } else {
      badges.appendChild(makeBadge('badge-deadline', '📅 ' + deadlineLabel(task.deadline)));
    }
    card.appendChild(badges);

    // footer with date
    var footer = document.createElement('div');
    footer.className = 'task-card-footer';
    var dateSpan = document.createElement('span');
    dateSpan.style.fontSize = '0.78rem';
    dateSpan.style.color = 'var(--text-muted)';
    dateSpan.textContent = formatDate(task.deadline);
    footer.appendChild(dateSpan);
    card.appendChild(footer);

    return card;
  }

  function iconBtn(emoji, label, extraClass, onClick) {
    if (typeof extraClass === 'function') { onClick = extraClass; extraClass = ''; }
    var btn = document.createElement('button');
    btn.className = 'icon-btn ' + (extraClass || '');
    btn.setAttribute('aria-label', label);
    btn.textContent = emoji;
    btn.addEventListener('click', onClick);
    return btn;
  }

  function makeBadge(cls, text) {
    var b = document.createElement('span');
    b.className = 'badge ' + cls;
    b.textContent = text;
    return b;
  }

  // ─── Dashboard ───────────────────────────────────────────
  function renderStats() {
    var total = tasks.length;
    var done = tasks.filter(function (t) { return t.completed; }).length;
    var overdue = tasks.filter(isOverdue).length;
    var pending = total - done;

    $('#statTotal').textContent = total;
    $('#statPending').textContent = pending;
    $('#statDone').textContent = done;
    $('#statOverdue').textContent = overdue;

    $('#heroStatTotal').textContent = total;
    $('#heroStatDone').textContent = done;
    $('#heroStatOverdue').textContent = overdue;

    var pct = total > 0 ? Math.round((done / total) * 100) : 0;
    $('#progressPct').textContent = pct + '%';
    $('#progressFill').style.width = pct + '%';
    var pb = document.querySelector('[role="progressbar"]');
    if (pb) { pb.setAttribute('aria-valuenow', pct); }

    // tab counts
    $('#countAll').textContent = total;
    $('#countPending').textContent = pending;
    $('#countCompleted').textContent = done;
    $('#countOverdue').textContent = overdue;
  }

  function renderAll() { renderStats(); renderTasks(); }

  // ─── CRUD ────────────────────────────────────────────────
  function toggleComplete(id) {
    var t = findTask(id);
    if (!t) return;
    t.completed = !t.completed;
    saveTasks();
    renderAll();
    toast(t.completed ? 'Task completed! 🎉' : 'Task marked as pending', t.completed ? 'success' : 'warning');
  }

  function deleteTask(id) {
    tasks = tasks.filter(function (t) { return t.id !== id; });
    saveTasks();
    renderAll();
    toast('Task deleted', 'error');
  }

  function findTask(id) {
    return tasks.filter(function (t) { return t.id === id; })[0];
  }

  // ─── Modal ───────────────────────────────────────────────
  var taskModal, confirmModal;

  function openModal(task) {
    var form = $('#taskForm');
    form.reset();
    clearErrors();

    if (task) {
      $('#modalTitle').textContent = 'Edit Task';
      $('#taskId').value = task.id;
      $('#taskTitle').value = task.title;
      $('#taskDesc').value = task.desc || '';
      $('#taskDeadline').value = task.deadline;
      $('#taskPriority').value = task.priority;
      $('#taskCategory').value = task.category;
    } else {
      $('#modalTitle').textContent = 'Add New Task';
      $('#taskId').value = '';
      $('#taskDeadline').value = addDays(1);
    }
    taskModal.hidden = false;
    setTimeout(function () { $('#taskTitle').focus(); }, 50);
  }

  function closeModal() {
    taskModal.hidden = true;
    clearErrors();
  }

  function clearErrors() {
    $('#errTitle').hidden = true;
    $('#errDeadline').hidden = true;
    $('#taskTitle').classList.remove('invalid');
    $('#taskDeadline').classList.remove('invalid');
  }

  function handleSubmit(e) {
    e.preventDefault();
    clearErrors();
    var valid = true;

    var title = $('#taskTitle').value.trim();
    if (!title) {
      $('#errTitle').textContent = 'Please enter a task title.';
      $('#errTitle').hidden = false;
      $('#taskTitle').classList.add('invalid');
      valid = false;
    }

    var deadline = $('#taskDeadline').value;
    if (!deadline) {
      $('#errDeadline').textContent = 'Please choose a deadline.';
      $('#errDeadline').hidden = false;
      $('#taskDeadline').classList.add('invalid');
      valid = false;
    }

    if (!valid) return;

    var id = $('#taskId').value;
    var data = {
      title: title,
      desc: $('#taskDesc').value.trim(),
      deadline: deadline,
      priority: $('#taskPriority').value,
      category: $('#taskCategory').value
    };

    if (id) {
      var t = findTask(id);
      if (t) { t.title = data.title; t.desc = data.desc; t.deadline = data.deadline; t.priority = data.priority; t.category = data.category; }
      toast('Task updated ✏️', 'success');
    } else {
      tasks.unshift({ id: uid(), title: data.title, desc: data.desc, deadline: data.deadline, priority: data.priority, category: data.category, completed: false, created: Date.now() });
      toast('Task added! 🎯', 'success');
    }
    saveTasks();
    closeModal();
    renderAll();
  }

  // ─── Confirm dialog ──────────────────────────────────────
  function confirmDelete(task) {
    pendingConfirm = function () { deleteTask(task.id); };
    $('#confirmText').textContent = 'Delete "' + task.title + '"? This cannot be undone.';
    confirmModal.hidden = false;
  }

  function closeConfirm() {
    confirmModal.hidden = true;
    pendingConfirm = null;
  }

  // ─── Toast ───────────────────────────────────────────────
  function toast(msg, type) {
    var el = document.createElement('div');
    el.className = 'toast ' + (type || 'success');
    var icon = type === 'error' ? '✕' : type === 'warning' ? '⚠' : '✓';
    el.innerHTML = '<span class="toast-icon">' + icon + '</span><span>' + msg + '</span>';
    toastContainer.appendChild(el);
    setTimeout(function () {
      el.classList.add('removing');
      setTimeout(function () { el.remove(); }, 250);
    }, 2800);
  }

  // ─── Reset demo ──────────────────────────────────────────
  function resetDemo() {
    pendingConfirm = function () {
      tasks = demoTasks.slice();
      saveTasks();
      renderAll();
      toast('Demo data restored ↺', 'success');
    };
    $('#confirmText').textContent = 'Reset all tasks to the original demo data? Your current tasks will be replaced.';
    confirmModal.hidden = false;
  }

  // ─── Nav / Scroll ────────────────────────────────────────
  function closeMobileNav() {
    navLinks.classList.remove('open');
    hamburger.classList.remove('open');
    hamburger.setAttribute('aria-expanded', 'false');
  }

  function setActiveNav() {
    var sections = ['hero', 'dashboard', 'tasks', 'about'];
    var scrollY = window.scrollY + 100;
    var current = 'hero';
    sections.forEach(function (id) {
      var el = document.getElementById(id);
      if (el && el.offsetTop <= scrollY) current = id;
    });
    document.querySelectorAll('.nav-link').forEach(function (link) {
      link.classList.toggle('active', link.dataset.section === current);
    });
    navbar.classList.toggle('scrolled', window.scrollY > 10);
  }

  // ─── Init ────────────────────────────────────────────────
  function init() {
    // cache DOM
    taskList = $('#taskList');
    emptyState = $('#emptyState');
    toastContainer = $('#toastContainer');
    loadingScreen = $('#loadingScreen');
    navbar = $('#navbar');
    navLinks = $('#navLinks');
    hamburger = $('#hamburger');
    taskModal = $('#taskModal');
    confirmModal = $('#confirmModal');

    tasks = loadTasks();

    // --- Button bindings ---
    $('#addTaskBtn').addEventListener('click', function () { openModal(null); });
    $('#heroAddBtn').addEventListener('click', function () { openModal(null); document.getElementById('tasks').scrollIntoView({ behavior: 'smooth' }); });
    $('#navAddBtn').addEventListener('click', function () { openModal(null); closeMobileNav(); });
    $('#emptyAddBtn').addEventListener('click', function () { openModal(null); });
    $('#resetBtn').addEventListener('click', resetDemo);

    // form
    $('#taskForm').addEventListener('submit', handleSubmit);
    $('#modalClose').addEventListener('click', closeModal);
    $('#modalCancel').addEventListener('click', closeModal);
    taskModal.addEventListener('click', function (e) { if (e.target === taskModal) closeModal(); });

    // confirm
    $('#confirmOk').addEventListener('click', function () { if (pendingConfirm) pendingConfirm(); closeConfirm(); });
    $('#confirmCancel').addEventListener('click', closeConfirm);
    confirmModal.addEventListener('click', function (e) { if (e.target === confirmModal) closeConfirm(); });

    // search
    $('#searchInput').addEventListener('input', function (e) {
      searchQuery = e.target.value.trim();
      renderTasks();
    });

    // filter tabs
    document.querySelectorAll('.filter-tab').forEach(function (tab) {
      tab.addEventListener('click', function () {
        document.querySelectorAll('.filter-tab').forEach(function (t) { t.classList.remove('active'); t.setAttribute('aria-selected', 'false'); });
        tab.classList.add('active');
        tab.setAttribute('aria-selected', 'true');
        currentFilter = tab.dataset.filter;
        renderTasks();
      });
    });

    // sort + priority filter
    $('#sortSelect').addEventListener('change', function (e) { currentSort = e.target.value; renderTasks(); });
    $('#priorityFilter').addEventListener('change', function (e) { currentPriorityFilter = e.target.value; renderTasks(); });

    // hamburger
    hamburger.addEventListener('click', function () {
      var open = navLinks.classList.toggle('open');
      hamburger.classList.toggle('open', open);
      hamburger.setAttribute('aria-expanded', open ? 'true' : 'false');
    });

    // nav link smooth scroll + close mobile
    document.querySelectorAll('.nav-link').forEach(function (link) {
      link.addEventListener('click', function () { closeMobileNav(); });
    });

    // keyboard: Esc closes modals
    document.addEventListener('keydown', function (e) {
      if (e.key === 'Escape') {
        if (!taskModal.hidden) closeModal();
        if (!confirmModal.hidden) closeConfirm();
        if (navLinks.classList.contains('open')) closeMobileNav();
      }
    });

    window.addEventListener('scroll', setActiveNav, { passive: true });

    // initial render
    renderAll();
    setActiveNav();

    // hide loading screen after short delay
    setTimeout(function () {
      loadingScreen.classList.add('hidden');
      setTimeout(function () { loadingScreen.style.display = 'none'; }, 400);
    }, 700);
  }

  if (document.readyState === 'loading') {
    document.addEventListener('DOMContentLoaded', init);
  } else {
    init();
  }
})();

