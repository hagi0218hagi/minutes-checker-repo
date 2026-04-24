<template>
  <div class="min-h-screen bg-gray-50 text-gray-900 font-sans p-4 sm:p-6 md:p-8">
    <div class="max-w-4xl mx-auto space-y-8">
      
      <!-- Header -->
      <header class="text-center space-y-2">
        <h1 class="text-3xl md:text-4xl font-extrabold tracking-tight text-indigo-700">
          AI議事録・タスク抽出チェッカー
        </h1>
        <p class="text-gray-500 text-sm md:text-base">
          AIが会議メモを解析し、「要約」「決定事項」「ToDo」を自動抽出します。
        </p>
      </header>

      <!-- Input Section -->
      <section class="bg-white rounded-2xl shadow-sm border border-gray-100 p-6 space-y-6">
        <!-- APIキー入力 -->
        <div>
          <label for="apiKey" class="block text-sm font-semibold text-gray-700 mb-1">
            Gemini APIキー
          </label>
          <div class="relative">
            <input
              id="apiKey"
              v-model="userApiKey"
              :type="showApiKey ? 'text' : 'password'"
              placeholder="AIzaSy..."
              class="w-full px-4 py-3 pr-12 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 transition-colors bg-gray-50 focus:bg-white font-mono text-sm"
            />
            <button
              @click="showApiKey = !showApiKey"
              type="button"
              class="absolute right-3 top-1/2 -translate-y-1/2 text-gray-400 hover:text-gray-600 transition-colors"
              :title="showApiKey ? 'APIキーを隠す' : 'APIキーを表示'"
            >
              <svg v-if="!showApiKey" class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 12a3 3 0 11-6 0 3 3 0 016 0z"/>
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M2.458 12C3.732 7.943 7.523 5 12 5c4.478 0 8.268 2.943 9.542 7-1.274 4.057-5.064 7-9.542 7-4.477 0-8.268-2.943-9.542-7z"/>
              </svg>
              <svg v-else class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13.875 18.825A10.05 10.05 0 0112 19c-4.478 0-8.268-2.943-9.543-7a9.97 9.97 0 011.563-3.029m5.858.908a3 3 0 114.243 4.243M9.878 9.878l4.242 4.242M9.88 9.88l-3.29-3.29m7.532 7.532l3.29 3.29M3 3l3.59 3.59m0 0A9.953 9.953 0 0112 5c4.478 0 8.268 2.943 9.543 7a10.025 10.025 0 01-4.132 5.411m0 0L21 21"/>
              </svg>
            </button>
          </div>
          <p class="mt-1 text-xs text-gray-400">
            APIキーは <a href="https://aistudio.google.com/app/apikey" target="_blank" rel="noopener noreferrer" class="text-indigo-500 hover:underline">Google AI Studio</a> で取得できます。入力内容はブラウザ外に送信されません。
          </p>
        </div>

        <!-- 会議タイトル入力 -->
        <div>
          <label for="meetingTitle" class="block text-sm font-semibold text-gray-700 mb-1">
            会議タイトル <span class="text-gray-400 font-normal">（任意）</span>
          </label>
          <input
            id="meetingTitle"
            v-model="meetingTitle"
            type="text"
            placeholder="例: 2026年4月 週次定例"
            class="w-full px-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 transition-colors bg-gray-50 focus:bg-white"
          />
        </div>

        <!-- 会議メモ入力 -->
        <div>
          <div class="flex justify-between items-center mb-1">
            <label for="transcript" class="block text-sm font-semibold text-gray-700">
              会議メモ / チャットログ
            </label>
            <button
              v-if="speechSupported"
              @click="toggleSpeech"
              type="button"
              :title="isRecording ? '録音を停止' : 'マイクで入力'"
              :class="[
                'flex items-center gap-1.5 px-3 py-1.5 rounded-lg text-sm font-medium transition-all',
                isRecording
                  ? 'bg-red-100 text-red-600 hover:bg-red-200 animate-pulse'
                  : 'bg-gray-100 text-gray-600 hover:bg-gray-200'
              ]"
            >
              <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 11a7 7 0 01-7 7m0 0a7 7 0 01-7-7m7 7v4m0 0H8m4 0h4m-4-8a3 3 0 01-3-3V5a3 3 0 116 0v6a3 3 0 01-3 3z"/>
              </svg>
              {{ isRecording ? '録音中... (停止)' : '音声入力' }}
            </button>
          </div>
          <textarea
            id="transcript"
            v-model="transcript"
            rows="10"
            placeholder="会議のメモや議事録、チャットの履歴をここに貼り付けてください（1000文字以上でも可）"
            class="w-full px-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 transition-colors resize-y bg-gray-50 focus:bg-white"
            :class="{ 'border-red-400 ring-2 ring-red-200': isRecording }"
          ></textarea>
          <p v-if="isRecording" class="mt-1 text-xs text-red-500 flex items-center gap-1">
            <span class="inline-block w-2 h-2 bg-red-500 rounded-full animate-ping"></span>
            話しかけてください。テキストエリアにリアルタイムで反映されます。
          </p>
        </div>

        <div class="flex flex-col sm:flex-row justify-between items-center bg-gray-50 p-4 rounded-xl border border-gray-100">
          <p class="text-xs text-gray-500 mb-4 sm:mb-0">
            モデル: <code>gemini-2.5-flash-preview-09-2025</code>
          </p>
          <button
            @click="analyzeText"
            :disabled="!isFormValid || isLoading || !userApiKey.trim()"
            class="w-full sm:w-auto px-6 py-3 bg-indigo-600 text-white font-medium rounded-lg shadow-sm hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2 disabled:opacity-50 disabled:cursor-not-allowed transition-all flex justify-center items-center space-x-2"
          >
            <svg v-if="isLoading" class="animate-spin -ml-1 mr-2 h-5 w-5 text-white" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
              <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
              <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
            </svg>
            <span v-if="isLoading">解析中...</span>
            <span v-else>解析を開始する</span>
          </button>
        </div>
        
        <!-- Error Message -->
        <div v-if="error" class="p-4 bg-red-50 border border-red-200 rounded-lg text-red-600 text-sm flex items-start">
          <svg class="w-5 h-5 mr-2 flex-shrink-0" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 8v4m0 4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z"></path></svg>
          {{ error }}
        </div>
      </section>

      <!-- Results Section -->
      <section v-if="hasResults" class="space-y-6 opacity-0 animate-fade-in-up">
        <div class="flex justify-between items-end mb-4 border-b border-gray-200 pb-2">
          <div>
            <h2 class="text-2xl font-bold text-gray-800">解析結果</h2>
            <p v-if="meetingTitle" class="text-sm text-indigo-600 font-medium mt-0.5">{{ meetingTitle }}</p>
          </div>
          <div class="flex items-center gap-2">
            <button
              @click="copyMarkdown"
              class="text-indigo-600 hover:text-indigo-800 bg-indigo-50 hover:bg-indigo-100 px-3 py-1.5 rounded-md text-sm font-medium flex items-center transition-colors"
            >
              <svg class="w-4 h-4 mr-1.5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 5H6a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2v-1M8 5a2 2 0 002 2h2a2 2 0 002-2M8 5a2 2 0 012-2h2a2 2 0 012 2m0 0h2a2 2 0 012 2v3m2 4H10m0 0l3-3m-3 3l3 3"></path></svg>
              コピー
            </button>
            <button
              @click="downloadMarkdown(results, null, meetingTitle || null)"
              class="text-emerald-600 hover:text-emerald-800 bg-emerald-50 hover:bg-emerald-100 px-3 py-1.5 rounded-md text-sm font-medium flex items-center transition-colors"
            >
              <svg class="w-4 h-4 mr-1.5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-4l-4 4m0 0l-4-4m4 4V4"></path></svg>
              ダウンロード
            </button>
          </div>
        </div>

        <!-- Summary Card -->
        <div class="bg-white rounded-2xl shadow-sm border border-gray-100 p-6 border-l-4 border-l-indigo-500 hover:shadow-md transition-shadow">
          <h3 class="text-lg font-bold text-gray-800 mb-3 flex items-center">
            <span class="bg-indigo-100 text-indigo-700 p-1.5 rounded-lg mr-2">
              <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 16h-1v-4h-1m1-4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z"></path></svg>
            </span>
            要約
          </h3>
          <p class="text-gray-700 leading-relaxed">{{ results.summary }}</p>
        </div>

        <!-- Decisions Card -->
        <div class="bg-white rounded-2xl shadow-sm border border-gray-100 p-6 border-l-4 border-l-green-500 hover:shadow-md transition-shadow">
          <h3 class="text-lg font-bold text-gray-800 mb-3 flex items-center">
            <span class="bg-green-100 text-green-700 p-1.5 rounded-lg mr-2">
              <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z"></path></svg>
            </span>
            決定事項
          </h3>
          <ul class="space-y-2">
            <li v-for="(decision, index) in results.decisions" :key="'dec-'+index" class="flex items-start">
              <span class="text-green-500 mr-2 mt-0.5 font-bold">•</span>
              <span class="text-gray-700">{{ decision }}</span>
            </li>
            <li v-if="results.decisions.length === 0" class="text-gray-400 italic text-sm">決定事項はありません。</li>
          </ul>
        </div>

        <!-- Tasks Card -->
        <div class="bg-white rounded-2xl shadow-sm border border-gray-100 p-6 border-l-4 border-l-amber-500 hover:shadow-md transition-shadow">
          <h3 class="text-lg font-bold text-gray-800 mb-4 flex items-center">
            <span class="bg-amber-100 text-amber-700 p-1.5 rounded-lg mr-2">
              <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5H7a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2V7a2 2 0 00-2-2h-2M9 5a2 2 0 002 2h2a2 2 0 002-2M9 5a2 2 0 012-2h2a2 2 0 012 2"></path></svg>
            </span>
            ToDo リスト
          </h3>
          <div class="space-y-3">
            <div v-for="(task, index) in results.tasks" :key="'task-'+index" class="flex items-start bg-gray-50 p-3 rounded-lg border border-gray-200 hover:border-amber-300 transition-colors" :class="{ 'opacity-60': checkedTasks[index] }">
              <div class="flex-shrink-0 pt-0.5">
                <input type="checkbox" v-model="checkedTasks[index]" @change="saveCheckedTasks" class="w-5 h-5 text-indigo-600 rounded border-gray-300 focus:ring-indigo-500 cursor-pointer" />
              </div>
              <div class="ml-3 flex-grow">
                <p class="text-gray-800 font-medium" :class="{ 'line-through text-gray-400': checkedTasks[index] }">{{ task.task }}</p>
                <div class="mt-2 flex flex-wrap gap-2 text-xs">
                  <span class="inline-flex items-center px-2 py-1 rounded-md font-medium bg-indigo-50 text-indigo-700 border border-indigo-100">
                    <svg class="w-3.5 h-3.5 mr-1" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M16 7a4 4 0 11-8 0 4 4 0 018 0zM12 14a7 7 0 00-7 7h14a7 7 0 00-7-7z"></path></svg>
                    {{ task.owner }}
                  </span>
                  <span class="inline-flex items-center px-2 py-1 rounded-md font-medium bg-red-50 text-red-700 border border-red-100">
                    <svg class="w-3.5 h-3.5 mr-1" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 7V3m8 4V3m-9 8h10M5 21h14a2 2 0 002-2V7a2 2 0 00-2-2H5a2 2 0 00-2 2v12a2 2 0 002 2z"></path></svg>
                    {{ task.due }}
                  </span>
                </div>
              </div>
            </div>
            <div v-if="results.tasks.length === 0" class="text-gray-400 italic text-sm">ToDoはありません。</div>
          </div>
        </div>
      </section>

      <!-- 解析履歴 Section -->
      <section v-if="history.length > 0" class="bg-white rounded-2xl shadow-sm border border-gray-100 p-6 space-y-4 animate-fade-in-up">
        <div class="flex justify-between items-center border-b border-gray-200 pb-3">
          <h2 class="text-xl font-bold text-gray-800 flex items-center">
            <svg class="w-5 h-5 mr-2 text-gray-500" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 8v4l3 3m6-3a9 9 0 11-18 0 9 9 0 0118 0z"></path></svg>
            解析履歴
          </h2>
          <div class="flex items-center gap-2">
            <button
              @click="downloadAllHistory"
              class="text-emerald-600 hover:text-emerald-800 bg-emerald-50 hover:bg-emerald-100 px-3 py-1.5 rounded-md text-sm font-medium flex items-center transition-colors"
            >
              <svg class="w-4 h-4 mr-1.5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-4l-4 4m0 0l-4-4m4 4V4"></path></svg>
              全件ダウンロード
            </button>
            <button
              @click="clearHistory"
              class="text-red-400 hover:text-red-600 bg-red-50 hover:bg-red-100 px-3 py-1.5 rounded-md text-sm font-medium flex items-center transition-colors"
            >
              <svg class="w-4 h-4 mr-1.5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"></path></svg>
              履歴を削除
            </button>
          </div>
        </div>

        <div class="space-y-3">
          <div
            v-for="item in history"
            :key="item.id"
            class="flex items-start justify-between bg-gray-50 rounded-xl p-4 border border-gray-100 hover:border-indigo-200 transition-colors"
          >
            <div class="flex-grow min-w-0 mr-4">
              <p class="text-xs text-gray-400 mb-1">{{ item.date }}</p>
              <p v-if="item.title" class="text-sm font-semibold text-indigo-600 truncate">{{ item.title }}</p>
              <p class="text-sm text-gray-700 truncate">{{ item.summary }}</p>
              <div class="mt-1 flex gap-3 text-xs text-gray-400">
                <span>決定事項 {{ item.decisions }}件</span>
                <span>ToDo {{ item.tasks }}件</span>
              </div>
            </div>
            <button
              @click="downloadMarkdown(item.data, item.date, item.title || null)"
              class="flex-shrink-0 text-emerald-600 hover:text-emerald-800 bg-emerald-50 hover:bg-emerald-100 px-3 py-1.5 rounded-md text-sm font-medium flex items-center transition-colors"
            >
              <svg class="w-4 h-4 mr-1" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-4l-4 4m0 0l-4-4m4 4V4"></path></svg>
              DL
            </button>
          </div>
        </div>
      </section>

      <!-- Omikuji Section -->
      <section class="bg-white rounded-2xl shadow-sm border border-pink-100 p-6 text-center space-y-4 hover:shadow-md transition-shadow animate-fade-in-up">
        <h2 class="text-xl font-bold text-gray-800 flex items-center justify-center">
          <span class="text-2xl mr-2">⛩️</span> AIからのタスク運勢おみくじ
        </h2>
        <p class="text-gray-500 text-sm">解析中や作業の合間に、これからの業務の運勢を占ってみましょう！</p>
        
        <div class="h-28 flex flex-col items-center justify-center bg-pink-50/50 rounded-xl mx-auto max-w-sm border border-pink-50 px-2 py-1">
          <div v-if="isDrawingFortune" class="animate-bounce text-5xl">🥠</div>
          <div v-else-if="currentFortune" class="flex flex-col items-center animate-fade-in-up w-full">
            <div class="text-4xl font-extrabold tracking-widest mb-1" :class="currentFortune.color">
              {{ currentFortune.rank }}
            </div>
            <div class="text-xs md:text-sm text-gray-700 font-medium px-4">
              {{ currentFortune.msg }}
            </div>
          </div>
          <div v-else class="text-5xl opacity-40 grayscale">🥠</div>
        </div>

        <button
          @click="drawFortune"
          :disabled="isDrawingFortune"
          class="px-8 py-3 bg-gradient-to-r from-red-400 to-rose-500 hover:from-red-500 hover:to-rose-600 text-white font-bold rounded-full shadow-sm transition-all focus:ring-2 focus:ring-rose-400 focus:ring-offset-2 disabled:opacity-50 disabled:cursor-not-allowed mx-auto transform hover:scale-105 active:scale-95"
        >
          {{ currentFortune ? 'もう一度引く' : 'おみくじを引く' }}
        </button>
      </section>
    </div>
    
    <!-- Toast -->
    <transition name="fade">
      <div
        v-if="showCopyNotification"
        class="fixed bottom-6 right-6 bg-gray-800 text-white px-5 py-3 rounded-xl shadow-2xl text-sm font-medium flex items-center z-50"
      >
        <span class="bg-green-500 rounded-full p-1 mr-3">
          <svg class="w-3 h-3 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="3" d="M5 13l4 4L19 7"></path></svg>
        </span>
        クリップボードにコピーしました！
      </div>
    </transition>
  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue';

// ========== State ==========
const transcript = ref('');
const meetingTitle = ref('');
const isLoading = ref(false);
const error = ref(null);
const results = ref(null);
const showCopyNotification = ref(false);

// ========== ToDoチェック状態 ==========
const CHECKED_KEY = 'meeting_checked_tasks';
const checkedTasks = ref(JSON.parse(localStorage.getItem(CHECKED_KEY) || '[]'));

const saveCheckedTasks = () => {
  localStorage.setItem(CHECKED_KEY, JSON.stringify(checkedTasks.value));
};

// 解析結果が変わったらチェック状態をリセット
watch(results, () => {
  checkedTasks.value = [];
  localStorage.removeItem(CHECKED_KEY);
});

// ========== APIキー State ==========
const userApiKey = ref(localStorage.getItem('gemini_api_key') || '');
const showApiKey = ref(false);

watch(userApiKey, (val) => {
  if (val.trim()) {
    localStorage.setItem('gemini_api_key', val.trim());
  } else {
    localStorage.removeItem('gemini_api_key');
  }
});

// ========== 履歴 State ==========
const HISTORY_KEY = 'meeting_history';
const loadHistory = () => {
  try {
    return JSON.parse(localStorage.getItem(HISTORY_KEY) || '[]');
  } catch {
    return [];
  }
};
const history = ref(loadHistory());

const saveToHistory = (data) => {
  const now = new Date();
  const dateStr = now.toLocaleString('ja-JP', { year: 'numeric', month: '2-digit', day: '2-digit', hour: '2-digit', minute: '2-digit' });
  const entry = {
    id: now.getTime(),
    date: dateStr,
    title: meetingTitle.value.trim() || null,
    summary: data.summary,
    decisions: data.decisions.length,
    tasks: data.tasks.length,
    data,
  };
  history.value = [entry, ...history.value].slice(0, 20); // 最大20件
  localStorage.setItem(HISTORY_KEY, JSON.stringify(history.value));
};

const clearHistory = () => {
  if (confirm('解析履歴をすべて削除しますか？')) {
    history.value = [];
    localStorage.removeItem(HISTORY_KEY);
  }
};

// ========== 音声入力 State ==========
const isRecording = ref(false);
const speechSupported = typeof window !== 'undefined' && ('SpeechRecognition' in window || 'webkitSpeechRecognition' in window);
let recognition = null;

if (speechSupported) {
  const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
  recognition = new SpeechRecognition();
  recognition.lang = 'ja-JP';
  recognition.continuous = true;      // 話し続けても止まらない
  recognition.interimResults = true;  // 途中結果もリアルタイム反映

  let finalTranscript = '';

  recognition.onstart = () => {
    finalTranscript = transcript.value; // 既存テキストを保持
    isRecording.value = true;
  };

  recognition.onresult = (event) => {
    let interim = '';
    for (let i = event.resultIndex; i < event.results.length; i++) {
      const t = event.results[i][0].transcript;
      if (event.results[i].isFinal) {
        finalTranscript += t;
      } else {
        interim += t;
      }
    }
    transcript.value = finalTranscript + interim;
  };

  recognition.onend = () => {
    isRecording.value = false;
  };

  recognition.onerror = (event) => {
    isRecording.value = false;
    if (event.error !== 'aborted') {
      error.value = `音声入力エラー: ${event.error}。マイクの許可を確認してください。`;
    }
  };
}

const toggleSpeech = () => {
  if (!recognition) return;
  if (isRecording.value) {
    recognition.stop();
  } else {
    error.value = null;
    recognition.start();
  }
};


const fortunes = [
  { rank: "大吉", msg: "プロジェクトは大成功の兆し！交渉もスムーズに進むでしょう。", color: "text-red-600" },
  { rank: "中吉", msg: "チームワークが冴え渡ります。新しいアイデアが採用されるかも。", color: "text-orange-500" },
  { rank: "吉", msg: "堅実な進展。ToDoを一つずつ潰すことで信頼が積み上がります。", color: "text-green-600" },
  { rank: "末吉", msg: "現状維持が吉。情報の共有漏れにだけ注意してください。", color: "text-blue-500" },
  { rank: "凶", msg: "スケジュールの再確認を。急ぎの案件が飛び込んでくる予感。", color: "text-purple-600" },
  { rank: "大凶", msg: "リスケ推奨。無理な納期は避け、まずは体勢を整えましょう。", color: "text-gray-500" }
];
const currentFortune = ref(null);
const isDrawingFortune = ref(false);

const drawFortune = () => {
  if (isDrawingFortune.value) return;
  isDrawingFortune.value = true;
  currentFortune.value = null;
  setTimeout(() => {
    currentFortune.value = fortunes[Math.floor(Math.random() * fortunes.length)];
    isDrawingFortune.value = false;
  }, 800);
};

// ========== Constants ==========
const MODEL_NAME = 'gemini-2.5-flash-preview-09-2025';
const SYSTEM_INSTRUCTION = "あなたはプロの議事録作成アシスタントです。入力されたテキストを分析し、重要な決定事項と、具体的なアクションアイテム（ToDo）を抽出してください。返り値は、指定されたJSONスキーマに従った純粋なJSONのみを返してください。";

// ========== Computed ==========
const isFormValid = computed(() => transcript.value.trim().length > 0);
const hasResults = computed(() => results.value !== null);

// ========== Markdown生成ヘルパー ==========
const buildMarkdown = (data, title = null) => {
  const { summary, decisions, tasks } = data;
  const lines = [];
  if (title) lines.push(`# ${title}\n`);
  lines.push("## 会議サマリー");
  lines.push(summary + "\n");
  lines.push("## 決定事項");
  decisions && decisions.length > 0
    ? decisions.forEach(d => lines.push(`- ${d}`))
    : lines.push("- なし");
  lines.push("");
  lines.push("## ToDoリスト");
  tasks && tasks.length > 0
    ? tasks.forEach(t => lines.push(`- [ ] **${t.task}** (担当: ${t.owner}, 期限: ${t.due})`))
    : lines.push("- なし");
  return lines.join('\n');
};

const makeTimestamp = () => {
  const now = new Date();
  return now.getFullYear().toString()
    + String(now.getMonth() + 1).padStart(2, '0')
    + String(now.getDate()).padStart(2, '0')
    + '_'
    + String(now.getHours()).padStart(2, '0')
    + String(now.getMinutes()).padStart(2, '0')
    + String(now.getSeconds()).padStart(2, '0');
};

// ========== Methods ==========
const fetchWithRetry = async (url, options, maxRetries = 3, baseDelay = 1000) => {
  let attempt = 0;
  while (attempt < maxRetries) {
    try {
      const response = await fetch(url, options);
      if (!response.ok) {
        if (response.status === 429 || response.status >= 500) throw new Error(`HTTP Error: ${response.status}`);
        let errorMessage = `API Error: ${response.status} ${response.statusText}`;
        try {
          const errorData = await response.json();
          if (errorData.error?.message) errorMessage = errorData.error.message;
        } catch(e) {}
        throw new Error(errorMessage, { cause: 'FATAL' });
      }
      return await response.json();
    } catch (err) {
      if (err.cause === 'FATAL' || attempt === maxRetries - 1) throw err;
      attempt++;
      const delay = baseDelay * Math.pow(2, attempt - 1);
      console.warn(`API呼び出しに失敗しました。${delay}ms後に再試行します... (${attempt}/${maxRetries}回目)`, err.message);
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
};

const analyzeText = async () => {
  if (!isFormValid.value) return;
  isLoading.value = true;
  error.value = null;
  results.value = null;
  drawFortune();

  const url = `https://generativelanguage.googleapis.com/v1beta/models/${MODEL_NAME}:generateContent?key=${userApiKey.value.trim()}`;
  const payload = {
    system_instruction: { parts: [{ text: SYSTEM_INSTRUCTION }] },
    contents: [{ parts: [{ text: transcript.value }] }],
    generationConfig: {
      response_mime_type: "application/json",
      response_schema: {
        type: "OBJECT",
        properties: {
          summary: { type: "STRING", description: "会議の簡潔な要約（150文字程度）" },
          decisions: { type: "ARRAY", items: { type: "STRING" }, description: "会議で決まったことの配列" },
          tasks: {
            type: "ARRAY",
            items: {
              type: "OBJECT",
              properties: {
                task: { type: "STRING", description: "ToDoの内容" },
                owner: { type: "STRING", description: "担当者" },
                due: { type: "STRING", description: "期限" }
              },
              required: ["task", "owner", "due"]
            }
          }
        },
        required: ["summary", "decisions", "tasks"]
      }
    }
  };

  try {
    const data = await fetchWithRetry(url, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(payload)
    });
    if (data.candidates && data.candidates.length > 0) {
      const parsed = JSON.parse(data.candidates[0].content.parts[0].text);
      results.value = parsed;
      saveToHistory(parsed); // 履歴に保存
    } else {
      throw new Error("予期しないAPIレスポンス形式です。解析結果が空でした。");
    }
  } catch (err) {
    console.error("Analysis Error:", err);
    error.value = err.message || "解析中にエラーが発生しました。ネットワーク接続やAPIキーが正しいか確認してください。";
  } finally {
    isLoading.value = false;
  }
};

/** 個別ダウンロード */
const downloadMarkdown = (data, label = null, title = null) => {
  if (!data) return;
  const blob = new Blob([buildMarkdown(data, title)], { type: 'text/markdown;charset=utf-8' });
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  const name = title || label;
  a.href = url;
  a.download = `meeting_${name ? name.replace(/[/:]/g, '').replace(/\s/g, '_') : makeTimestamp()}.md`;
  a.click();
  URL.revokeObjectURL(url);
};

/** 全履歴を1ファイルにまとめてダウンロード */
const downloadAllHistory = () => {
  if (history.value.length === 0) return;
  const lines = [];
  history.value.forEach((item, i) => {
    if (i > 0) lines.push('\n---\n');
    lines.push(`<!-- ${item.date} -->`);
    lines.push(buildMarkdown(item.data, item.title || null));
  });
  const blob = new Blob([lines.join('\n')], { type: 'text/markdown;charset=utf-8' });
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  a.download = `meeting_history_${makeTimestamp()}.md`;
  a.click();
  URL.revokeObjectURL(url);
};

/** クリップボードコピー */
const copyMarkdown = async () => {
  if (!results.value) return;
  try {
    await navigator.clipboard.writeText(buildMarkdown(results.value, meetingTitle.value.trim() || null));
    showCopyNotification.value = true;
    setTimeout(() => { showCopyNotification.value = false; }, 3000);
  } catch (err) {
    alert('クリップボードへのコピーに失敗しました。ブラウザの設定をご確認ください。');
  }
};
</script>

<style scoped>
.animate-fade-in-up {
  animation: fadeInUp 0.5s ease-out forwards;
}
@keyframes fadeInUp {
  from { opacity: 0; transform: translateY(10px); }
  to { opacity: 1; transform: translateY(0); }
}
.fade-enter-active, .fade-leave-active { transition: opacity 0.3s ease; }
.fade-enter-from, .fade-leave-to { opacity: 0; }
</style>
