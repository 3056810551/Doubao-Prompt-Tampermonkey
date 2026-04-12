// ==UserScript==
// @name         Doubao Skill Bar Append Button
// @namespace    https://tampermonkey.net/
// @version      0.3.0
// @description  在技能栏“更多”按钮后追加一个可持久化的提示词面板
// @match        *://*.doubao.com/*
// @match        *://doubao.com/*
// @run-at       document-idle
// @grant        none
// ==/UserScript==

(function () {
  "use strict";

  const CUSTOM_BUTTON_ID = "tm-custom-skill-button";
  const PANEL_ID = "tm-prompt-panel";
  const STYLE_ID = "tm-prompt-panel-style";
  const TOAST_ID = "tm-prompt-panel-toast";
  const STORAGE_KEY = "tm_doubao_prompt_library_v1";
  const SETTINGS_KEY = "tm_doubao_prompt_settings_v1";
  const CUSTOM_BUTTON_TEXT = "提示词";
  const PANEL_WIDTH = 376;
  const PANEL_GAP = 12;
  const CUSTOM_BUTTON_ICON = '<svg viewBox="0 0 24 24" aria-hidden="true"><path d="M6.75 3.75H17.25C19.0449 3.75 20.5 5.20507 20.5 7V12C20.5 13.7949 19.0449 15.25 17.25 15.25H15.6816L12.9443 18.3955C12.4461 18.968 11.5539 18.968 11.0557 18.3955L8.31836 15.25H6.75C4.95508 15.25 3.5 13.7949 3.5 12V7C3.5 5.20508 4.95508 3.75 6.75 3.75Z" stroke="currentColor" stroke-width="1.75" stroke-linejoin="round" fill="none"/><path d="M8 8.25H16M8 11.75H12.75" stroke="currentColor" stroke-width="1.75" stroke-linecap="round"/></svg>';
  const DEFAULT_PROMPTS = [
    { id: createId(), name: "结构化笔记", text: "把下面内容整理成结构化笔记，保留重点并输出标题与要点。" },
    { id: createId(), name: "自然改写", text: "把这段话改写得更自然、更口语化，但不要改变原意。" },
    { id: createId(), name: "提炼结论", text: "帮我提炼这段内容的核心结论，并给出 3 个可执行建议。" },
  ];
  const state = {
    isOpen: false,
    search: "",
    prompts: loadPrompts(),
    autoSend: loadSettings().autoSend,
    isAdding: false,
    addNameDraft: "",
    addDraft: "",
    editingId: null,
    editNameDraft: "",
    editDraft: "",
    focusTarget: "search",
    draggingId: null,
    dragHandleId: null,
  };

  function createId() {
    return `prompt_${Date.now()}_${Math.random().toString(36).slice(2, 8)}`;
  }

  function loadPrompts() {
    try {
      const raw = window.localStorage.getItem(STORAGE_KEY);
      if (!raw) return DEFAULT_PROMPTS;
      const parsed = JSON.parse(raw);
      if (!Array.isArray(parsed)) return DEFAULT_PROMPTS;
      const sanitized = parsed
        .filter((item) => item && typeof item.text === "string")
        .map((item) => {
          const text = item.text.trim();
          const name = typeof item.name === "string" && item.name.trim() ? item.name.trim() : derivePromptName(text);
          return { id: typeof item.id === "string" ? item.id : createId(), name, text };
        })
        .filter((item) => item.text);
      return sanitized.length ? sanitized : DEFAULT_PROMPTS;
    } catch (error) {
      console.error("[tm-prompt-panel] load prompts failed", error);
      return DEFAULT_PROMPTS;
    }
  }

  function savePrompts() {
    window.localStorage.setItem(STORAGE_KEY, JSON.stringify(state.prompts));
  }

  function loadSettings() {
    try {
      const raw = window.localStorage.getItem(SETTINGS_KEY);
      if (!raw) return { autoSend: false };
      const parsed = JSON.parse(raw);
      return { autoSend: Boolean(parsed?.autoSend) };
    } catch (error) {
      console.error("[tm-prompt-panel] load settings failed", error);
      return { autoSend: false };
    }
  }

  function saveSettings() {
    window.localStorage.setItem(SETTINGS_KEY, JSON.stringify({ autoSend: state.autoSend }));
  }

  function derivePromptName(text) {
    const normalized = String(text || "").replace(/\s+/g, " ").trim();
    if (!normalized) return "未命名提示词";
    return normalized.length > 18 ? `${normalized.slice(0, 18)}...` : normalized;
  }

  function escapeHtml(text) {
    return text.replaceAll("&", "&amp;").replaceAll("<", "&lt;").replaceAll(">", "&gt;").replaceAll('"', "&quot;").replaceAll("'", "&#39;");
  }

  function injectStyles() {
    if (document.getElementById(STYLE_ID)) return;
    const style = document.createElement("style");
    style.id = STYLE_ID;
    style.textContent = `#${PANEL_ID}{position:fixed;z-index:2147483647;display:none;flex-direction:column;width:${PANEL_WIDTH}px;height:520px;max-width:calc(100vw - 24px);max-height:calc(100vh - 24px);overflow:hidden;color:#0f172a;background:rgba(255,255,255,.94);border:1px solid rgba(226,232,240,.92);border-radius:22px;box-shadow:0 24px 80px rgba(15,23,42,.16);backdrop-filter:blur(20px)}#${PANEL_ID}.is-open{display:flex}#${PANEL_ID} *,#${PANEL_ID} *::before,#${PANEL_ID} *::after,#${TOAST_ID} *,#${TOAST_ID} *::before,#${TOAST_ID} *::after{box-sizing:border-box}.tm-panel-header{display:flex;flex-direction:column;gap:12px;padding:16px 16px 12px;border-bottom:1px solid rgba(241,245,249,.95);background:radial-gradient(circle at top right,rgba(99,102,241,.1),transparent 34%),linear-gradient(180deg,rgba(255,255,255,.98),rgba(248,250,252,.9))}.tm-panel-search{display:flex;align-items:center;gap:10px;height:46px;padding:0 14px;border:1px solid rgba(203,213,225,.9);border-radius:14px;background:rgba(255,255,255,.96);transition:border-color .18s ease,box-shadow .18s ease}.tm-panel-search:focus-within{border-color:rgba(99,102,241,.72);box-shadow:0 0 0 4px rgba(99,102,241,.08)}.tm-panel-search input{flex:1;min-width:0;border:0;outline:0;background:transparent;color:#0f172a;font-size:14px}.tm-panel-search input::placeholder{color:#94a3b8}.tm-panel-search svg{flex:none;width:18px;height:18px;color:#64748b}.tm-panel-toolbar{display:flex;align-items:center;justify-content:space-between;gap:12px}.tm-panel-count{color:#64748b;font-size:13px}.tm-switch{display:inline-flex;align-items:center;gap:8px;color:#334155;font-size:13px;white-space:nowrap;cursor:pointer;user-select:none}.tm-switch input{display:none}.tm-switch-track{position:relative;width:36px;height:22px;border-radius:999px;background:#cbd5e1;transition:background .18s ease}.tm-switch-track::after{content:"";position:absolute;top:3px;left:3px;width:16px;height:16px;border-radius:50%;background:#fff;box-shadow:0 1px 3px rgba(15,23,42,.22);transition:transform .18s ease}.tm-switch input:checked+.tm-switch-track{background:#4f46e5}.tm-switch input:checked+.tm-switch-track::after{transform:translateX(14px)}.tm-panel-list{flex:1;min-height:0;overflow:auto;padding:12px}.tm-panel-empty{padding:30px 18px;color:#64748b;text-align:center;font-size:14px}.tm-prompt-stack{display:flex;flex-direction:column;gap:10px}.tm-prompt-card,.tm-editor-card{display:flex;align-items:center;gap:10px;padding:12px;border:1px solid rgba(226,232,240,.95);border-radius:18px;background:rgba(255,255,255,.98)}.tm-prompt-card:hover{border-color:rgba(165,180,252,.95);box-shadow:0 8px 24px rgba(99,102,241,.08)}.tm-prompt-card.is-dragging{opacity:.48;border-color:rgba(99,102,241,.55);box-shadow:none}.tm-prompt-card.is-drop-target{border-color:rgba(99,102,241,.72);box-shadow:0 0 0 3px rgba(99,102,241,.1)}.tm-prompt-main{flex:1;min-width:0;padding:4px 2px;border:0;background:transparent;text-align:left;cursor:pointer;color:inherit}.tm-prompt-row{display:flex;align-items:center;gap:6px;min-width:0}.tm-prompt-index{flex:none;min-width:28px;color:#94a3b8;font-size:12px;font-weight:700;letter-spacing:.04em}.tm-prompt-name{color:#0f172a;font-size:15px;font-weight:600;line-height:1.4;white-space:nowrap;overflow:hidden;text-overflow:ellipsis}.tm-prompt-actions{display:flex;align-items:center;gap:8px;flex:none}.tm-action-btn,.tm-footer-btn,.tm-editor-btn{display:inline-flex;align-items:center;justify-content:center;gap:6px;min-width:34px;height:34px;padding:0 10px;border:1px solid rgba(203,213,225,.95);border-radius:12px;background:rgba(255,255,255,.95);color:#334155;cursor:pointer;transition:border-color .18s ease,box-shadow .18s ease,color .18s ease,background .18s ease}.tm-action-btn:hover,.tm-footer-btn:hover,.tm-editor-btn:hover{color:#4338ca;border-color:rgba(99,102,241,.72);background:rgba(238,242,255,.92)}.tm-action-btn[data-kind="danger"]:hover,.tm-editor-btn[data-kind="danger"]:hover{color:#dc2626;border-color:rgba(248,113,113,.68);background:rgba(254,242,242,.95)}.tm-action-btn[data-action="drag"]{cursor:grab}.tm-action-btn svg,.tm-footer-btn svg,.tm-editor-btn svg{width:16px;height:16px}.tm-editor{display:flex;flex-direction:column;gap:10px;width:100%}.tm-editor-input,.tm-editor textarea{width:100%;border:1px solid rgba(203,213,225,.95);border-radius:14px;outline:0;font-size:14px;line-height:1.65;color:#0f172a;background:rgba(255,255,255,.98)}.tm-editor-input{height:44px;padding:0 14px}.tm-editor textarea{min-height:100px;resize:vertical;padding:12px 14px}.tm-editor-input:focus,.tm-editor textarea:focus{border-color:rgba(99,102,241,.72);box-shadow:0 0 0 4px rgba(99,102,241,.08)}.tm-editor-actions{display:flex;justify-content:flex-end;gap:8px}.tm-panel-footer{display:flex;gap:10px;padding:12px;border-top:1px solid rgba(241,245,249,.95);background:rgba(248,250,252,.82)}.tm-footer-btn{flex:1;height:42px;border-radius:14px;font-size:14px;font-weight:600}.tm-footer-btn[data-kind="primary"],.tm-editor-btn[data-kind="primary"]{color:#fff;border-color:#4f46e5;background:linear-gradient(180deg,#6366f1,#4f46e5)}.tm-footer-btn[data-kind="primary"]:hover,.tm-editor-btn[data-kind="primary"]:hover{color:#fff;border-color:#4338ca;background:linear-gradient(180deg,#4f46e5,#4338ca)}#${TOAST_ID}{position:fixed;left:50%;bottom:24px;z-index:2147483647;display:none;max-width:calc(100vw - 32px);padding:10px 14px;border-radius:999px;color:#fff;background:rgba(15,23,42,.88);box-shadow:0 10px 24px rgba(15,23,42,.24);transform:translateX(-50%);font-size:13px}#${TOAST_ID}.is-open{display:block}`;
    document.head.appendChild(style);
  }

  function getIcon(name) {
    const icons = {
      search: '<svg viewBox="0 0 24 24" fill="none"><circle cx="11" cy="11" r="6.5" stroke="currentColor" stroke-width="1.8"></circle><path d="M16 16L20 20" stroke="currentColor" stroke-width="1.8" stroke-linecap="round"></path></svg>',
      add: '<svg viewBox="0 0 24 24" fill="none"><path d="M12 5V19M5 12H19" stroke="currentColor" stroke-width="1.8" stroke-linecap="round"></path></svg>',
      drag: '<svg viewBox="0 0 24 24" fill="none"><path d="M9 6.5A1.25 1.25 0 1 1 9 9a1.25 1.25 0 0 1 0-2.5ZM15 6.5A1.25 1.25 0 1 1 15 9a1.25 1.25 0 0 1 0-2.5ZM9 11A1.25 1.25 0 1 1 9 13.5 1.25 1.25 0 0 1 9 11ZM15 11A1.25 1.25 0 1 1 15 13.5 1.25 1.25 0 0 1 15 11ZM9 15.5A1.25 1.25 0 1 1 9 18a1.25 1.25 0 0 1 0-2.5ZM15 15.5A1.25 1.25 0 1 1 15 18a1.25 1.25 0 0 1 0-2.5Z" fill="currentColor"></path></svg>',
      edit: '<svg viewBox="0 0 24 24" fill="none"><path d="M4 20L8.2 18.9L18.2 8.9C18.98 8.12 18.98 6.86 18.2 6.08L17.92 5.8C17.14 5.02 15.88 5.02 15.1 5.8L5.1 15.8L4 20Z" stroke="currentColor" stroke-width="1.8" stroke-linejoin="round"></path></svg>',
      delete: '<svg viewBox="0 0 24 24" fill="none"><path d="M5 7H19M9 7V5.5C9 4.67 9.67 4 10.5 4H13.5C14.33 4 15 4.67 15 5.5V7M8 7L8.7 18.2C8.76 19.19 9.58 19.96 10.57 19.96H13.43C14.42 19.96 15.24 19.19 15.3 18.2L16 7" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"></path><path d="M10 10.5V16M14 10.5V16" stroke="currentColor" stroke-width="1.8" stroke-linecap="round"></path></svg>',
      up: '<svg viewBox="0 0 24 24" fill="none"><path d="M12 18V6M12 6L7.5 10.5M12 6L16.5 10.5" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"></path></svg>',
      down: '<svg viewBox="0 0 24 24" fill="none"><path d="M12 6V18M12 18L7.5 13.5M12 18L16.5 13.5" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"></path></svg>',
      close: '<svg viewBox="0 0 24 24" fill="none"><path d="M7 7L17 17M17 7L7 17" stroke="currentColor" stroke-width="1.8" stroke-linecap="round"></path></svg>',
      save: '<svg viewBox="0 0 24 24" fill="none"><path d="M5 12.5L9.2 16.7L19 7" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"></path></svg>',
    };
    return icons[name] || "";
  }

  function createButton(className) {
    const button = document.createElement("button");
    button.id = CUSTOM_BUTTON_ID;
    button.type = "button";
    button.className = className;
    button.setAttribute("data-dbx-name", "button");
    button.innerHTML = `${CUSTOM_BUTTON_ICON}<div class="min-w-0 truncate"><div class="flex items-center gap-4">${CUSTOM_BUTTON_TEXT}</div></div>`;
    button.addEventListener("click", togglePanel);
    return button;
  }

  function createPanel() {
    const existing = document.getElementById(PANEL_ID);
    if (existing) return existing;
    const panel = document.createElement("div");
    panel.id = PANEL_ID;
    panel.addEventListener("click", (event) => event.stopPropagation());
    document.body.appendChild(panel);
    return panel;
  }

  function createToast() {
    const existing = document.getElementById(TOAST_ID);
    if (existing) return existing;
    const toast = document.createElement("div");
    toast.id = TOAST_ID;
    document.body.appendChild(toast);
    return toast;
  }

  function ensureUi() {
    injectStyles();
    createPanel();
    createToast();
  }

  function showToast(message) {
    const toast = createToast();
    toast.textContent = message;
    toast.classList.add("is-open");
    window.clearTimeout(showToast.timerId);
    showToast.timerId = window.setTimeout(() => toast.classList.remove("is-open"), 1800);
  }

  showToast.timerId = 0;

  function appendButton() {
    if (document.getElementById(CUSTOM_BUTTON_ID)) return;
    const moreButton = document.querySelector('button[data-test-id="skill_bar_button_more"]');
    if (!moreButton) return;
    ensureUi();
    const anchor = moreButton.parentElement?.tagName === "DIV" ? moreButton.parentElement : moreButton;
    anchor.insertAdjacentElement("afterend", createButton(moreButton.className));
  }

  function getFilteredPrompts() {
    const keyword = state.search.trim().toLowerCase();
    if (!keyword) return state.prompts;
    return state.prompts.filter((item) => `${item.name} ${item.text}`.toLowerCase().includes(keyword));
  }

  function setPanelPosition() {
    const button = document.getElementById(CUSTOM_BUTTON_ID);
    const panel = document.getElementById(PANEL_ID);
    if (!button || !panel) return;
    const rect = button.getBoundingClientRect();
    const panelWidth = Math.min(PANEL_WIDTH, window.innerWidth - 24);
    const left = Math.max(12, Math.min(rect.right - panelWidth, window.innerWidth - panelWidth - 12));
    panel.style.width = `${panelWidth}px`;
    panel.style.left = `${left}px`;
    panel.style.top = "12px";
    panel.classList.add("is-open");
    const panelHeight = panel.offsetHeight;
    let top = rect.top - panelHeight - PANEL_GAP;
    if (top < 12) top = Math.min(rect.bottom + PANEL_GAP, window.innerHeight - panelHeight - 12);
    panel.style.top = `${Math.max(12, top)}px`;
  }

  function renderPanel() {
    ensureUi();
    const panel = document.getElementById(PANEL_ID);
    if (!panel) return;
    const filtered = getFilteredPrompts();
    panel.innerHTML = `<div class="tm-panel-header"><label class="tm-panel-search"><input id="tm-prompt-search" type="text" placeholder="搜索提示词..." value="${escapeHtml(state.search)}">${getIcon("search")}</label><div class="tm-panel-toolbar"><div class="tm-panel-count">共 ${state.prompts.length} 条，显示 ${filtered.length} 条</div><label class="tm-switch" title="点选提示词后自动发送"><input id="tm-auto-send-toggle" type="checkbox" ${state.autoSend ? "checked" : ""}><span class="tm-switch-track"></span><span>自动发送</span></label></div></div><div class="tm-panel-list">${renderPromptList(filtered)}</div><div class="tm-panel-footer"><button type="button" class="tm-footer-btn" data-action="close-panel">${getIcon("close")}关闭</button><button type="button" class="tm-footer-btn" data-kind="primary" data-action="toggle-add">${getIcon("add")}${state.isAdding ? "收起新增" : "新增提示词"}</button></div>`;
    bindPanelEvents();
    if (state.isOpen) {
      panel.classList.add("is-open");
      setPanelPosition();
    } else {
      panel.classList.remove("is-open");
    }
  }

  function renderPromptList(prompts) {
    const addEditor = state.isAdding ? `<div class="tm-editor-card"><div class="tm-editor"><input id="tm-add-name-input" class="tm-editor-input" type="text" placeholder="提示词名称" value="${escapeHtml(state.addNameDraft)}"><textarea id="tm-add-textarea" placeholder="输入新的提示词内容...">${escapeHtml(state.addDraft)}</textarea><div class="tm-editor-actions"><button type="button" class="tm-editor-btn" data-action="cancel-add">取消</button><button type="button" class="tm-editor-btn" data-kind="primary" data-action="save-add">${getIcon("save")}保存</button></div></div></div>` : "";
    if (!prompts.length) return `<div class="tm-prompt-stack">${addEditor}<div class="tm-panel-empty">没有匹配到提示词，试试修改搜索词或新增一个。</div></div>`;
    return `<div class="tm-prompt-stack">${addEditor}${prompts.map((item, index) => {
      if (state.editingId === item.id) {
        return `<div class="tm-editor-card" data-id="${item.id}"><div class="tm-editor"><input class="tm-editor-input" data-role="edit-name-input" type="text" placeholder="提示词名称" value="${escapeHtml(state.editNameDraft)}"><textarea data-role="edit-textarea">${escapeHtml(state.editDraft)}</textarea><div class="tm-editor-actions"><button type="button" class="tm-editor-btn" data-action="cancel-edit" data-id="${item.id}">取消</button><button type="button" class="tm-editor-btn" data-kind="primary" data-action="save-edit" data-id="${item.id}">${getIcon("save")}保存</button></div></div></div>`;
      }
      return `<div class="tm-prompt-card${state.draggingId === item.id ? " is-dragging" : ""}" data-id="${item.id}" draggable="true"><button type="button" class="tm-prompt-main" data-action="apply-prompt" data-id="${item.id}"><div class="tm-prompt-row"><div class="tm-prompt-index">${String(index + 1).padStart(2, "0")}</div><div class="tm-prompt-name">${escapeHtml(item.name)}</div></div></button><div class="tm-prompt-actions"><button type="button" class="tm-action-btn" data-action="drag" data-drag-handle="true" data-id="${item.id}" title="拖拽排序">${getIcon("drag")}</button><button type="button" class="tm-action-btn" data-action="edit" data-id="${item.id}" title="编辑">${getIcon("edit")}</button><button type="button" class="tm-action-btn" data-kind="danger" data-action="delete" data-id="${item.id}" title="删除">${getIcon("delete")}</button></div></div>`;
    }).join("")}</div>`;
  }

  function bindPanelEvents() {
    const panel = document.getElementById(PANEL_ID);
    if (!panel) return;
    const searchInput = panel.querySelector("#tm-prompt-search");
    if (searchInput) {
      searchInput.addEventListener("input", (event) => {
        state.search = event.target.value;
        state.focusTarget = "search";
        renderPanel();
      });
    }
    const autoSendToggle = panel.querySelector("#tm-auto-send-toggle");
    if (autoSendToggle) {
      autoSendToggle.addEventListener("change", (event) => {
        state.autoSend = event.target.checked;
        saveSettings();
        renderPanel();
      });
    }
    const addNameInput = panel.querySelector("#tm-add-name-input");
    if (addNameInput) addNameInput.addEventListener("input", (event) => { state.addNameDraft = event.target.value; });
    const addTextarea = panel.querySelector("#tm-add-textarea");
    if (addTextarea) addTextarea.addEventListener("input", (event) => { state.addDraft = event.target.value; });
    const editNameInput = panel.querySelector('[data-role="edit-name-input"]');
    if (editNameInput) editNameInput.addEventListener("input", (event) => { state.editNameDraft = event.target.value; });
    const editTextarea = panel.querySelector('[data-role="edit-textarea"]');
    if (editTextarea) editTextarea.addEventListener("input", (event) => { state.editDraft = event.target.value; });
    panel.querySelectorAll("[data-drag-handle]").forEach((element) => {
      element.addEventListener("pointerdown", () => {
        state.dragHandleId = element.dataset.id || null;
      });
      element.addEventListener("pointerup", () => {
        state.dragHandleId = null;
      });
      element.addEventListener("pointerleave", () => {
        state.dragHandleId = null;
      });
    });
    panel.querySelectorAll("[data-action]").forEach((element) => element.addEventListener("click", handlePanelAction));
    panel.querySelectorAll(".tm-prompt-card[draggable='true']").forEach((element) => {
      element.addEventListener("dragstart", handleDragStart);
      element.addEventListener("dragover", handleDragOver);
      element.addEventListener("dragleave", handleDragLeave);
      element.addEventListener("drop", handleDrop);
      element.addEventListener("dragend", handleDragEnd);
    });
    queueMicrotask(() => {
      const target = state.focusTarget === "add"
        ? panel.querySelector("#tm-add-name-input")
        : state.focusTarget === "edit"
          ? panel.querySelector('[data-role="edit-name-input"]')
          : panel.querySelector("#tm-prompt-search");
      if (!target) return;
      target.focus();
      if (typeof target.setSelectionRange === "function") {
        const length = target.value.length;
        target.setSelectionRange(length, length);
      }
    });
  }

  function handlePanelAction(event) {
    event.stopPropagation();
    const action = event.currentTarget.dataset.action;
    const id = event.currentTarget.dataset.id || "";
    if (action === "drag") return;
    if (action === "close-panel") return closePanel();
    if (action === "toggle-add") {
      state.isAdding = !state.isAdding;
      state.focusTarget = state.isAdding ? "add" : "search";
      if (!state.isAdding) {
        state.addNameDraft = "";
        state.addDraft = "";
      }
      return renderPanel();
    }
    if (action === "cancel-add") {
      state.isAdding = false;
      state.addNameDraft = "";
      state.addDraft = "";
      state.focusTarget = "search";
      return renderPanel();
    }
    if (action === "save-add") {
      const name = state.addNameDraft.trim();
      const text = state.addDraft.trim();
      if (!name) return showToast("提示词名称不能为空");
      if (!text) return showToast("提示词内容不能为空");
      state.prompts.unshift({ id: createId(), name, text });
      state.addNameDraft = "";
      state.addDraft = "";
      state.isAdding = false;
      state.focusTarget = "search";
      savePrompts();
      renderPanel();
      return showToast("已新增提示词");
    }
    if (action === "edit") {
      const current = state.prompts.find((item) => item.id === id);
      if (!current) return;
      state.editingId = id;
      state.editNameDraft = current.name || derivePromptName(current.text);
      state.editDraft = current.text;
      state.isAdding = false;
      state.focusTarget = "edit";
      return renderPanel();
    }
    if (action === "cancel-edit") {
      state.editingId = null;
      state.editNameDraft = "";
      state.editDraft = "";
      state.focusTarget = "search";
      return renderPanel();
    }
    if (action === "save-edit") {
      const name = state.editNameDraft.trim();
      const text = state.editDraft.trim();
      if (!name) return showToast("提示词名称不能为空");
      if (!text) return showToast("提示词内容不能为空");
      state.prompts = state.prompts.map((item) => item.id === id ? { ...item, name, text } : item);
      state.editingId = null;
      state.editNameDraft = "";
      state.editDraft = "";
      state.focusTarget = "search";
      savePrompts();
      renderPanel();
      return showToast("已保存修改");
    }
    if (action === "delete") {
      const current = state.prompts.find((item) => item.id === id);
      const confirmed = window.confirm(
        `确定删除提示词「${current?.name || "未命名提示词"}」吗？此操作无法撤销。`,
      );
      if (!confirmed) return;
      state.prompts = state.prompts.filter((item) => item.id !== id);
      if (state.editingId === id) {
        state.editingId = null;
        state.editNameDraft = "";
        state.editDraft = "";
      }
      state.focusTarget = "search";
      savePrompts();
      renderPanel();
      return showToast("已删除提示词");
    }
    if (action === "apply-prompt") return applyPromptToDoubao(id);
  }

  function movePrompt(id, direction) {
    const index = state.prompts.findIndex((item) => item.id === id);
    const nextIndex = index + direction;
    if (index < 0 || nextIndex < 0 || nextIndex >= state.prompts.length) return;
    const list = [...state.prompts];
    const [current] = list.splice(index, 1);
    list.splice(nextIndex, 0, current);
    state.prompts = list;
    savePrompts();
    renderPanel();
  }

  function movePromptToTarget(draggingId, targetId, insertAfter) {
    const fromIndex = state.prompts.findIndex((item) => item.id === draggingId);
    const targetIndex = state.prompts.findIndex((item) => item.id === targetId);
    if (fromIndex < 0 || targetIndex < 0 || fromIndex === targetIndex) return;
    const list = [...state.prompts];
    const [dragged] = list.splice(fromIndex, 1);
    const adjustedTargetIndex = fromIndex < targetIndex ? targetIndex - 1 : targetIndex;
    const insertIndex = insertAfter ? adjustedTargetIndex + 1 : adjustedTargetIndex;
    list.splice(insertIndex, 0, dragged);
    state.prompts = list;
    state.draggingId = null;
    state.dragHandleId = null;
    savePrompts();
    renderPanel();
  }

  function clearDropTargets() {
    document.querySelectorAll(".tm-prompt-card.is-drop-target").forEach((element) => {
      element.classList.remove("is-drop-target");
    });
  }

  function handleDragStart(event) {
    const card = event.currentTarget;
    const id = card.dataset.id || "";
    if (!id || state.dragHandleId !== id) {
      event.preventDefault();
      return;
    }
    state.draggingId = id;
    card.classList.add("is-dragging");
    if (event.dataTransfer) {
      event.dataTransfer.effectAllowed = "move";
      event.dataTransfer.setData("text/plain", id);
    }
  }

  function handleDragOver(event) {
    if (!state.draggingId) return;
    const target = event.currentTarget;
    if ((target.dataset.id || "") === state.draggingId) return;
    event.preventDefault();
    clearDropTargets();
    target.classList.add("is-drop-target");
    if (event.dataTransfer) event.dataTransfer.dropEffect = "move";
  }

  function handleDragLeave(event) {
    event.currentTarget.classList.remove("is-drop-target");
  }

  function handleDrop(event) {
    event.preventDefault();
    const target = event.currentTarget;
    const targetId = target.dataset.id || "";
    const draggingId = state.draggingId || event.dataTransfer?.getData("text/plain") || "";
    clearDropTargets();
    if (!draggingId || !targetId || draggingId === targetId) return;
    const rect = target.getBoundingClientRect();
    const insertAfter = event.clientY > rect.top + rect.height / 2;
    movePromptToTarget(draggingId, targetId, insertAfter);
  }

  function handleDragEnd() {
    state.draggingId = null;
    state.dragHandleId = null;
    clearDropTargets();
    document.querySelectorAll(".tm-prompt-card.is-dragging").forEach((element) => {
      element.classList.remove("is-dragging");
    });
  }

  function applyPromptToDoubao(id) {
    const prompt = state.prompts.find((item) => item.id === id);
    if (!prompt) return;
    const composer = findComposerElement();
    if (!composer) return showToast("未找到豆包输入框");
    const currentText = getComposerText(composer).trim();
    const nextText = currentText ? `${prompt.text}\n\n${currentText}` : prompt.text;
    setComposerText(composer, nextText, prompt.text.length);
    closePanel();
    if (state.autoSend) {
      return window.setTimeout(() => {
        const sent = triggerSend(composer);
        showToast(sent ? "已插入并发送" : "已插入，未找到发送按钮");
      }, 80);
    }
    showToast("已插入到输入框最前面");
  }
  function isVisible(element) {
    if (!element || !(element instanceof HTMLElement)) return false;
    const rect = element.getBoundingClientRect();
    const style = window.getComputedStyle(element);
    return rect.width > 0 && rect.height > 0 && style.display !== "none" && style.visibility !== "hidden" && style.opacity !== "0";
  }

  function isInCustomUi(element) {
    return Boolean(element?.closest?.(`#${PANEL_ID}`) || element?.closest?.(`#${CUSTOM_BUTTON_ID}`));
  }

  function scoreComposerCandidate(element) {
    const rect = element.getBoundingClientRect();
    const tagName = element.tagName.toLowerCase();
    const hintText = [
      element.getAttribute("placeholder") || "",
      element.getAttribute("aria-label") || "",
      element.getAttribute("data-placeholder") || "",
      element.getAttribute("role") || "",
      typeof element.className === "string" ? element.className : "",
    ].join(" ").toLowerCase();
    let score = 0;
    if (tagName === "textarea") score += 40;
    if (element.isContentEditable) score += 34;
    if (hintText.includes("发送") || hintText.includes("输入") || hintText.includes("message")) score += 24;
    if (rect.width > 260) score += 20;
    if (rect.bottom > window.innerHeight * 0.5) score += 18;
    if (rect.top > window.innerHeight * 0.28) score += 10;
    if (element.closest("footer,form,main")) score += 8;
    return score;
  }

  function findComposerElement() {
    const selector = ["textarea", '[contenteditable="true"]', '[contenteditable="plaintext-only"]', '[role="textbox"]'].join(",");
    const candidates = Array.from(document.querySelectorAll(selector))
      .filter((element) => isVisible(element))
      .filter((element) => !isInCustomUi(element))
      .map((element) => ({ element, score: scoreComposerCandidate(element) }))
      .sort((a, b) => b.score - a.score);
    return candidates[0]?.element || null;
  }

  function getComposerText(element) {
    if ("value" in element && typeof element.value === "string") return element.value;
    if (element.isContentEditable) return element.innerText || element.textContent || "";
    return "";
  }

  function setNativeValue(element, value) {
    const prototype = Object.getPrototypeOf(element);
    const descriptor = Object.getOwnPropertyDescriptor(prototype, "value");
    if (descriptor?.set) descriptor.set.call(element, value);
    else element.value = value;
  }

  function dispatchTextEvents(element) {
    element.dispatchEvent(new Event("input", { bubbles: true }));
    element.dispatchEvent(new Event("change", { bubbles: true }));
  }

  function setComposerText(element, value, caretPosition) {
    element.focus();
    if ("value" in element && typeof element.value === "string") {
      setNativeValue(element, value);
      dispatchTextEvents(element);
      if (typeof element.setSelectionRange === "function") element.setSelectionRange(caretPosition, caretPosition);
      return;
    }
    if (element.isContentEditable) {
      element.textContent = value;
      dispatchTextEvents(element);
      placeCaretAtOffset(element, caretPosition);
    }
  }

  function placeCaretAtOffset(element, offset) {
    const selection = window.getSelection();
    if (!selection) return;
    const range = document.createRange();
    const walker = document.createTreeWalker(element, NodeFilter.SHOW_TEXT);
    let remaining = offset;
    let node = walker.nextNode();
    while (node) {
      const length = node.textContent?.length || 0;
      if (remaining <= length) {
        range.setStart(node, remaining);
        range.collapse(true);
        selection.removeAllRanges();
        selection.addRange(range);
        return;
      }
      remaining -= length;
      node = walker.nextNode();
    }
    range.selectNodeContents(element);
    range.collapse(false);
    selection.removeAllRanges();
    selection.addRange(range);
  }

  function scoreSendButton(button, composer) {
    if (!isVisible(button) || isInCustomUi(button) || button.disabled) return -Infinity;
    const text = [
      button.textContent || "",
      button.getAttribute("aria-label") || "",
      button.getAttribute("title") || "",
      button.getAttribute("data-testid") || "",
      typeof button.className === "string" ? button.className : "",
    ].join(" ").toLowerCase();
    const buttonRect = button.getBoundingClientRect();
    const composerRect = composer.getBoundingClientRect();
    let score = 0;
    if (text.includes("发送") || text.includes("send")) score += 48;
    if (text.includes("submit")) score += 20;
    if (button.closest("form") && composer.closest("form") === button.closest("form")) score += 24;
    if (buttonRect.left >= composerRect.left - 24) score += 8;
    if (Math.abs(buttonRect.bottom - composerRect.bottom) < 160) score += 8;
    return score;
  }

  function findSendButton(composer) {
    const ranked = Array.from(document.querySelectorAll("button"))
      .map((button) => ({ button, score: scoreSendButton(button, composer) }))
      .filter((item) => Number.isFinite(item.score) && item.score > 0)
      .sort((a, b) => b.score - a.score);
    return ranked[0]?.button || null;
  }

  function triggerSend(composer) {
    const sendButton = findSendButton(composer);
    if (sendButton) {
      sendButton.click();
      return true;
    }
    composer.focus();
    const options = { key: "Enter", code: "Enter", keyCode: 13, which: 13, bubbles: true, cancelable: true };
    composer.dispatchEvent(new KeyboardEvent("keydown", options));
    composer.dispatchEvent(new KeyboardEvent("keypress", options));
    composer.dispatchEvent(new KeyboardEvent("keyup", options));
    return false;
  }

  function openPanel() {
    state.isOpen = true;
    state.focusTarget = "search";
    renderPanel();
  }

  function closePanel() {
    state.isOpen = false;
    state.isAdding = false;
    state.editingId = null;
    state.addNameDraft = "";
    state.addDraft = "";
    state.editNameDraft = "";
    state.editDraft = "";
    renderPanel();
  }

  function togglePanel(event) {
    if (event) event.stopPropagation();
    if (state.isOpen) closePanel();
    else openPanel();
  }

  function handleDocumentClick(event) {
    if (!state.isOpen) return;
    const panel = document.getElementById(PANEL_ID);
    const button = document.getElementById(CUSTOM_BUTTON_ID);
    if (panel?.contains(event.target) || button?.contains(event.target)) return;
    closePanel();
  }

  function handleKeydown(event) {
    if (event.key === "Escape" && state.isOpen) closePanel();
  }

  function initObservers() {
    const observer = new MutationObserver(() => appendButton());
    observer.observe(document.body, { childList: true, subtree: true });
  }

  function initGlobalEvents() {
    document.addEventListener("click", handleDocumentClick, true);
    window.addEventListener("resize", () => state.isOpen && setPanelPosition());
    window.addEventListener("scroll", () => state.isOpen && setPanelPosition(), true);
    document.addEventListener("keydown", handleKeydown, true);
  }

  function init() {
    ensureUi();
    appendButton();
    initObservers();
    initGlobalEvents();
  }

  init();
})();
