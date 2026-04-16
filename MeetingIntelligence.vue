<template>
  <div class="min-h-screen bg-gray-50 text-gray-900 font-sans p-4 sm:p-6 md:p-8">
    <div class="max-w-4xl mx-auto space-y-8">
      
      <!-- Header -->
      <header class="text-center space-y-2">
        <h1 class="text-3xl md:text-4xl font-extrabold tracking-tight text-indigo-700">
          Meeting Intelligence
        </h1>
        <p class="text-gray-500 text-sm md:text-base">
          AIが会議メモを解析し、「要約」「決定事項」「ToDo」を自動抽出します。
        </p>
      </header>

      <!-- Input Section -->
      <section class="bg-white rounded-2xl shadow-sm border border-gray-100 p-6 space-y-6">
        <!-- 会議メモ入力のみに変更 -->
        <div>
          <label for="transcript" class="block text-sm font-semibold text-gray-700 mb-1">
            会議メモ / チャットログ
          </label>
          <textarea
            id="transcript"
            v-model="transcript"
            rows="10"
            placeholder="会議のメモや議事録、チャットの履歴をここに貼り付けてください（1000文字以上でも可）"
            class="w-full px-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 transition-colors resize-y bg-gray-50 focus:bg-white"
          ></textarea>
        </div>

        <div class="flex flex-col sm:flex-row justify-between items-center bg-gray-50 p-4 rounded-xl border border-gray-100">
          <p class="text-xs text-gray-500 mb-4 sm:mb-0">
            モデル: <code>gemini-2.5-flash-preview-09-2025</code>
          </p>
          <button
            @click="analyzeText"
            :disabled="!isFormValid || isLoading"
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
          <h2 class="text-2xl font-bold text-gray-800">解析結果</h2>
          <button
            @click="copyMarkdown"
            class="text-indigo-600 hover:text-indigo-800 bg-indigo-50 hover:bg-indigo-100 px-3 py-1.5 rounded-md text-sm font-medium flex items-center transition-colors"
          >
            <svg class="w-4 h-4 mr-1.5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 5H6a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2v-1M8 5a2 2 0 002 2h2a2 2 0 002-2M8 5a2 2 0 012-2h2a2 2 0 012 2m0 0h2a2 2 0 012 2v3m2 4H10m0 0l3-3m-3 3l3 3"></path></svg>
            Markdownコピー
          </button>
        </div>

        <!-- Summary Card -->
        <div class="bg-white rounded-2xl shadow-sm border border-gray-100 p-6 border-l-4 border-l-indigo-500 hover:shadow-md transition-shadow">
          <h3 class="text-lg font-bold text-gray-800 mb-3 flex items-center">
            <span class="bg-indigo-100 text-indigo-700 p-1.5 rounded-lg mr-2">
              <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 16h-1v-4h-1m1-4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z"></path></svg>
            </span>
            要約
          </h3>
          <p class="text-gray-700 leading-relaxed">
            {{ results.summary }}
          </p>
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
            <div v-for="(task, index) in results.tasks" :key="'task-'+index" class="flex items-start bg-gray-50 p-3 rounded-lg border border-gray-200 hover:border-amber-300 transition-colors">
              <div class="flex-shrink-0 pt-0.5">
                <input type="checkbox" class="w-5 h-5 text-indigo-600 rounded border-gray-300 focus:ring-indigo-500 cursor-pointer" />
              </div>
              <div class="ml-3 flex-grow">
                <p class="text-gray-800 font-medium">{{ task.task }}</p>
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
    </div>
    
    <!-- Copy Notification Toast -->
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
import { ref, computed } from 'vue';

// ========== State ==========
const transcript = ref('');
const isLoading = ref(false);
const error = ref(null);
const results = ref(null);
const showCopyNotification = ref(false);

// 環境変数からAPIキーを取得 (Viteの import.meta.env を使用)
const apiKey = import.meta.env.VITE_GEMINI_API_KEY || '';

// ========== Constants ==========
const MODEL_NAME = 'gemini-2.5-flash-preview-09-2025';
const SYSTEM_INSTRUCTION = "あなたはプロの議事録作成アシスタントです。入力されたテキストを分析し、重要な決定事項と、具体的なアクションアイテム（ToDo）を抽出してください。返り値は、指定されたJSONスキーマに従った純粋なJSONのみを返してください。";

// ========== Computed ==========
const isFormValid = computed(() => {
  return transcript.value.trim().length > 0;
});

const hasResults = computed(() => {
  return results.value !== null;
});

// ========== Methods ==========

/**
 * 指数バックオフを用いたリトライ付きfetch
 * @param {string} url - API URL
 * @param {object} options - fetchのオプション
 * @param {number} maxRetries - 最大リトライ回数
 * @param {number} baseDelay - 初回リトライの待機時間（ミリ秒）
 */
const fetchWithRetry = async (url, options, maxRetries = 3, baseDelay = 1000) => {
  let attempt = 0;
  
  while (attempt < maxRetries) {
    try {
      const response = await fetch(url, options);
      
      if (!response.ok) {
        // HTTP 429(Rate Limit) または 500系エラーの場合のみリトライさせる
        if (response.status === 429 || response.status >= 500) {
          throw new Error(`HTTP Error: ${response.status}`);
        }
        
        // 認証エラーやリクエスト不正などの致命的エラー（400, 401, 403, 404）は即座にエラーとしてスロー
        let errorMessage = `API Error: ${response.status} ${response.statusText}`;
        try {
          const errorData = await response.json();
          if (errorData.error && errorData.error.message) {
            errorMessage = errorData.error.message;
          }
        } catch(e) {
          // JSONパースエラー時はフォールバックのメッセージを利用
        }
        throw new Error(errorMessage, { cause: 'FATAL' });
      }
      
      return await response.json();
    } catch (err) {
      if (err.cause === 'FATAL' || attempt === maxRetries - 1) {
        throw err; // 再試行すべきでないエラー、もしくは最大回数到達で最終的にスロー
      }
      
      attempt++;
      // 指数バックオフ: 1000ms -> 2000ms -> 4000ms ...
      const delay = baseDelay * Math.pow(2, attempt - 1);
      console.warn(`API呼び出しに失敗しました。${delay}ms後に再試行します... (${attempt}/${maxRetries}回目)`, err.message);
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
};

/**
 * 解析メイン処理
 */
const analyzeText = async () => {
  if (!isFormValid.value) return;

  isLoading.value = true;
  error.value = null;
  results.value = null;

  const url = `https://generativelanguage.googleapis.com/v1beta/models/${MODEL_NAME}:generateContent?key=${apiKey.trim()}`;

  // Structured Output(JSON Schema)を指定したGemini APIリクエストボディ
  const payload = {
    system_instruction: {
      parts: [{ text: SYSTEM_INSTRUCTION }]
    },
    contents: [
      {
        parts: [{ text: transcript.value }]
      }
    ],
    generationConfig: {
      response_mime_type: "application/json",
      response_schema: {
        type: "OBJECT",
        properties: {
          summary: {
            type: "STRING",
            description: "会議の簡潔な要約（150文字程度）"
          },
          decisions: {
            type: "ARRAY",
            items: { type: "STRING" },
            description: "会議で決まったことの配列"
          },
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
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(payload)
    });

    if (data.candidates && data.candidates.length > 0) {
      // JSONスキーマを指定しているため、テキストパーツからそのままJSONパース可能
      const responseText = data.candidates[0].content.parts[0].text;
      results.value = JSON.parse(responseText);
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

/**
 * Markdown形式でクリップボードへコピー
 */
const copyMarkdown = async () => {
  if (!results.value) return;

  const { summary, decisions, tasks } = results.value;

  const mdLines = [];
  mdLines.push("# 会議サマリー");
  mdLines.push(summary + "\n");
  
  mdLines.push("## 決定事項");
  if (decisions && decisions.length > 0) {
    decisions.forEach(d => mdLines.push(`- ${d}`));
  } else {
    mdLines.push("- なし");
  }
  mdLines.push("");

  mdLines.push("## ToDoリスト");
  if (tasks && tasks.length > 0) {
    tasks.forEach(t => {
      mdLines.push(`- [ ] **${t.task}** (担当: ${t.owner}, 期限: ${t.due})`);
    });
  } else {
    mdLines.push("- なし");
  }

  const markdownText = mdLines.join('\n');

  try {
    await navigator.clipboard.writeText(markdownText);
    showCopyNotification.value = true;
    setTimeout(() => {
      showCopyNotification.value = false;
    }, 3000);
  } catch (err) {
    console.error('Failed to copy text: ', err);
    alert('クリップボードへのコピーに失敗しました。ブラウザの設定をご確認ください。');
  }
};
</script>

<style scoped>
.animate-fade-in-up {
  animation: fadeInUp 0.5s ease-out forwards;
}

@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translateY(10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
