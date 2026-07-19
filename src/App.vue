<template>
  <div class="app-container">
    <!-- 顶部生成成功浮动 Toast -->
    <transition name="fade">
      <div v-if="showSuccessToast" class="top-success-toast">
        <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round">
          <polyline points="20 6 9 17 4 12"></polyline>
        </svg>
        <span>论文大纲与学术撰写方案生成成功！</span>
      </div>
    </transition>

    <!-- 右上角常驻分享按钮 -->
    <button class="floating-share-btn" @click="showShareGuide = true">
      <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="share-icon">
        <circle cx="18" cy="5" r="3"></circle>
        <circle cx="6" cy="12" r="3"></circle>
        <circle cx="18" cy="19" r="3"></circle>
        <line x1="8.59" y1="13.51" x2="15.42" y2="17.49"></line>
        <line x1="15.41" y1="6.51" x2="8.59" y2="10.49"></line>
      </svg>
      <span>分享</span>
    </button>

    <!-- 顶部 App Header (已安全移除未登录提示栏) -->
    <header>
      <h1>{{ appTitle }}</h1>
      <p>智能 AI 学术引擎 · 毕业论文/开题报告/文献综述/期刊论文一键设计</p>
    </header>

    <!-- 活跃动态 -->
    <UserTicker />

    <!-- 核心卡片 / 结果与历史记录展示 -->
    <main class="glass-card input-group">
      <div class="card-tabs">
        <button class="tab-btn" :class="{ active: !showHistory }" @click="showHistory = false">
          论文写作生成
        </button>
        <button class="tab-btn" :class="{ active: showHistory }" @click="showHistory = true">
          历史记录 ({{ historyList.length }})
        </button>
      </div>

      <!-- 本地历史记录视图 -->
      <div v-if="showHistory" class="history-view">
        <div class="history-header">
          <span>本地论文历史</span>
          <button v-if="historyList.length > 0" class="clear-all-btn" @click="clearAllHistory">清空全部</button>
        </div>

        <div v-if="historyList.length === 0" class="empty-state">
          <p>暂无论文大纲与学术撰写历史记录</p>
        </div>

        <div v-else class="history-grid">
          <div v-for="item in historyList" :key="item.id" class="history-card">
            <div class="h-card-header">
              <span class="h-card-style">{{ item.paperType }} · {{ item.degree }}</span>
              <span class="h-card-time">{{ item.timestamp }}</span>
            </div>
            
            <div class="h-card-body">
              <div class="h-card-nomad-title">
                <span class="h-city-tag">题目: {{ item.destination }}</span>
                <span class="h-score-badge">学术评分: {{ getAverageScore(item) }}</span>
              </div>

              <p class="h-card-excerpt"><strong>大纲摘要：</strong>{{ cleanExcerpt(item.output) }}</p>
            </div>

            <div class="h-card-actions">
              <button class="h-action-btn load-btn" @click="selectHistoryItem(item)">
                加载详情
              </button>
              <button class="h-action-btn delete-btn" @click="deleteHistoryRecord(item.id)">
                删除
              </button>
            </div>
          </div>
        </div>
      </div>

      <div v-else>
        <!-- 主输入区域 (参考图一精准复刻 UI 布局) -->
        <div v-if="!result" class="divination-setup">
          
          <!-- 参考图一 1：23 种论文/报告类型胶囊选择区 -->
          <div class="paper-types-container">
            <div class="paper-types-grid">
              <button 
                v-for="type in paperTypeList" 
                :key="type"
                class="paper-type-pill"
                :class="{ active: activePaperType === type }"
                @click="activePaperType = type"
              >
                <svg xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" class="pill-icon">
                  <path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"></path>
                  <polyline points="14 2 14 8 20 8"></polyline>
                  <line x1="16" y1="13" x2="8" y2="13"></line>
                  <line x1="16" y1="17" x2="8" y2="17"></line>
                  <line x1="10" y1="9" x2="8" y2="9"></line>
                </svg>
                <span>{{ type }}</span>
              </button>
            </div>
          </div>

          <!-- 参考图一 2：核心题目与写作要求输入框 (含标题推荐优化) -->
          <div class="textarea-wrapper-custom">
            <textarea 
              v-model="destination"
              class="topic-textarea-custom"
              placeholder="输入题目和写作要求等，越详细越好(可输入10万字)"
              rows="4"
            ></textarea>
            <div class="textarea-footer-actions">
              <button class="recommend-title-btn" @click="handleRecommendTitle">
                <svg xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                  <line x1="8" y1="6" x2="21" y2="6"></line>
                  <line x1="8" y1="12" x2="21" y2="12"></line>
                  <line x1="8" y1="18" x2="21" y2="18"></line>
                  <line x1="3" y1="6" x2="3.01" y2="6"></line>
                  <line x1="3" y1="12" x2="3.01" y2="12"></line>
                  <line x1="3" y1="18" x2="3.01" y2="18"></line>
                </svg>
                <span>标题推荐优化</span>
              </button>
            </div>
          </div>

          <!-- 参考图一 3：细化单参数配置区 (学历/字数/模型/大纲/语言) -->
          <div class="academic-options-panel">
            
            <!-- 学历选择 -->
            <div class="option-row">
              <span class="option-row-label">学历：</span>
              <div class="radio-group-custom">
                <label v-for="deg in degreeList" :key="deg" class="radio-item-custom">
                  <input type="radio" :value="deg" v-model="selectedDegree" />
                  <span class="radio-label-text">{{ deg }}</span>
                </label>
              </div>
            </div>

            <!-- 字数选择 -->
            <div class="option-row">
              <span class="option-row-label">字数：</span>
              <div class="radio-group-custom">
                <label v-for="wc in wordCountList" :key="wc" class="radio-item-custom">
                  <input type="radio" :value="wc" v-model="selectedWordCount" />
                  <span class="radio-label-text">{{ wc }}</span>
                </label>
              </div>
            </div>

            <!-- 模型选择 -->
            <div class="option-row">
              <span class="option-row-label">模型：</span>
              <div class="radio-group-custom">
                <label class="radio-item-custom">
                  <input type="radio" value="标准模型" v-model="selectedModel" />
                  <span class="radio-label-text">标准模型</span>
                </label>
                <label class="radio-item-custom highlight-model">
                  <input type="radio" value="5.0(Genius Writer)" v-model="selectedModel" />
                  <span class="radio-label-text">5.0(Genius Writer) <small style="color: #ec4899;">(DeepSeek V4 学术加强版)</small></span>
                </label>
              </div>
            </div>

            <!-- 大纲层级选择 -->
            <div class="option-row">
              <span class="option-row-label">大纲：</span>
              <div class="radio-group-custom">
                <label v-for="ol in outlineLevelList" :key="ol" class="radio-item-custom">
                  <input type="radio" :value="ol" v-model="selectedOutline" />
                  <span class="radio-label-text">{{ ol }}</span>
                </label>
              </div>
            </div>

            <!-- 语言选择 -->
            <div class="option-row">
              <span class="option-row-label">语言：</span>
              <div class="radio-group-custom">
                <label v-for="lang in languageList" :key="lang" class="radio-item-custom">
                  <input type="radio" :value="lang" v-model="selectedLanguage" />
                  <span class="radio-label-text">{{ lang }}</span>
                </label>
              </div>
            </div>
          </div>

          <!-- 参考图一 4：生成大纲主提交按钮 -->
          <button 
            class="action-btn generate-outline-btn" 
            :disabled="!destination.trim() || loading" 
            @click="handleGenerate"
          >
            <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <line x1="8" y1="6" x2="21" y2="6"></line>
              <line x1="8" y1="12" x2="21" y2="12"></line>
              <line x1="8" y1="18" x2="21" y2="18"></line>
              <line x1="3" y1="6" x2="3.01" y2="6"></line>
              <line x1="3" y1="12" x2="3.01" y2="12"></line>
              <line x1="3" y1="18" x2="3.01" y2="18"></line>
            </svg>
            <span>{{ loading ? '学术导师与大纲生成中...' : '生成大纲' }}</span>
          </button>

          <!-- 异常提示 -->
          <div v-if="errorMsg" class="error-banner">
            {{ errorMsg }}
          </div>
        </div>

        <!-- 结果展现 -->
        <div v-else class="divination-result">
          <!-- 学术学士帽打卡交互板块 -->
          <div class="stamp-section">
            <div class="stamp-canvas">
              <svg 
                class="stamp-svg" 
                :class="{ stamping: isStamping }" 
                @click="stampCheckin" 
                viewBox="0 0 160 160"
              >
                <!-- 学术学士帽像素印章 -->
                <circle cx="80" cy="80" r="70" fill="rgba(168, 85, 247, 0.08)" stroke="#a855f7" stroke-width="4" stroke-dasharray="6,3" />
                <polygon points="80,30 140,55 80,80 20,55" fill="#a855f7"/>
                <polygon points="45,67 45,95 80,115 115,95 115,67 80,85" fill="#9333ea"/>
                <line x1="140" y1="55" x2="140" y2="90" stroke="#f43f5e" stroke-width="3"/>
                <circle cx="140" cy="93" r="4" fill="#f43f5e"/>
                <text x="80" y="142" font-size="12" font-weight="bold" fill="#a855f7" text-anchor="middle">学士帽打卡</text>
              </svg>
              <!-- 漂浮在看动效 (严格无 Emoji) -->
              <transition-group name="float-up">
                <span 
                  v-for="item in floatingItems" 
                  :key="item.id" 
                  class="floating-merit"
                  :style="{ transform: `translate(${item.x}px, ${item.y}px)`, color: '#a855f7' }"
                >
                  {{ item.text }}
                </span>
              </transition-group>
            </div>
            <div class="merit-counter-display">
              累计学术论文打卡：<strong style="color: #a855f7;">{{ totalCheckin }}</strong> 次
              <p class="wood-fish-tip">点击上方学士帽印章，听学术铜铃声为严谨治学打卡</p>
            </div>
          </div>

          <!-- 单轨学术指标多维评估看板 (AI共识判定) -->
          <div v-if="aiScores" class="comparison-dashboard">
            <h3 class="dashboard-title">学术质量多维评估 (AI共识判定)</h3>
            <div class="comparison-grid">
              <div v-for="metric in metricsList" :key="metric.key" class="comparison-row">
                <div class="metric-info">
                  <span class="metric-label">{{ metric.label }}</span>
                  <span class="metric-scores-text">
                    指数值: <strong style="color: var(--accent-color)">{{ aiScores[metric.key] }} / 5</strong>
                  </span>
                </div>
                <div class="comparison-bars">
                  <div class="bar-container">
                    <div class="bar-bg">
                      <div class="bar-fill ai-fill" :style="{ width: aiScores[metric.key] * 20 + '%' }"></div>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <div class="result-header">
            <div class="result-title-group">
              <span class="result-title">学术论文大纲与撰写指南</span>
              <span class="success-badge">生成成功</span>
            </div>
            <div class="button-actions">
              <button class="icon-btn" @click="copyText">
                {{ copied ? '已复制全文' : '复制结果' }}
              </button>
              <button class="icon-btn highlight" @click="showShareCard = true">
                生成分享卡片
              </button>
              <button class="icon-btn restart-btn" @click="resetReview">
                重新生成
              </button>
            </div>
          </div>

          <!-- 加载中骨架屏 -->
          <div v-if="loading" class="skeleton">
            <div class="skeleton-line" style="width: 80%"></div>
            <div class="skeleton-line" style="width: 95%"></div>
            <div class="skeleton-line" style="width: 60%"></div>
            <div class="skeleton-line" style="width: 75%"></div>
          </div>

          <!-- 渲染回复 -->
          <div class="ai-response-wrapper">
            <div class="output-content scroll-box" style="text-align: left;">{{ displayResultText }}</div>
          </div>
        </div>
      </div>
    </main>

    <!-- 演示案例区组件 (模块三：30 条经典学术论文案例展示) -->
    <DemoShowcase @use-sample="handleUseSample" />

    <!-- 底部隐私与服务条款链接 -->
    <footer class="footer-links">
      <button class="footer-link-btn" @click="showPrivacy = true">Privacy Policy</button>
      <button class="footer-link-btn" @click="showTerms = true">Terms of Service</button>
      <button class="footer-link-btn" @click="showContact = true">Contact Us</button>
    </footer>

    <!-- 隐私政策弹窗 -->
    <div v-if="showPrivacy" class="modal-overlay" @click.self="showPrivacy = false">
      <div class="modal-content">
        <h3>Privacy Policy</h3>
        <div class="modal-text-content modal-scroll-area">
          <p>我们非常重视您的学术隐私。您在本应用中输入的论文题目、研究方向与大纲参数仅用于实时大模型分析与大纲生成，我们不会在服务器端进行永久存储或记录。</p>
          <p>为了记录您的学术历史、免费额度和打卡数据，本应用会在您的浏览器本地（localStorage）保存相关状态。</p>
        </div>
        <button class="modal-btn" @click="showPrivacy = false">关闭</button>
      </div>
    </div>

    <!-- 服务条款弹窗 -->
    <div v-if="showTerms" class="modal-overlay" @click.self="showTerms = false">
      <div class="modal-content">
        <h3>Terms of Service</h3>
        <div class="modal-text-content modal-scroll-area">
          <p>欢迎使用我们的 AI 论文写作与学术大纲生成专家微应用。生成的大纲与参考文献仅供学术开题与论文架构参考，请遵守学术道德与诚信规范。</p>
        </div>
        <button class="modal-btn" @click="showTerms = false">关闭</button>
      </div>
    </div>

    <!-- 联系我们弹窗 (自适应高度 + weixin.png & dingtalk.png 展示) -->
    <div v-if="showContact" class="modal-overlay" @click.self="showContact = false">
      <div class="modal-content contact-modal-content">
        <h3>Contact Us</h3>
        <div class="modal-text-content contact-card-body">
          <p>如果您在使用过程中遇到任何问题，或有合作意向，可以通过以下方式联系我们：</p>
          <div class="contact-qr-container">
            <div class="contact-qr-card">
              <img :src="weixinImg" alt="微信联系" class="contact-qr-img" />
              <span class="contact-qr-label">微信联系</span>
            </div>
            <div class="contact-qr-card">
              <img :src="dingtalkImg" alt="钉钉交流" class="contact-qr-img" />
              <span class="contact-qr-label">钉钉交流</span>
            </div>
          </div>
          <p class="contact-email">反馈邮箱: <span style="color: var(--primary-color);">us@wuxian.xyz</span></p>
        </div>
        <button class="modal-btn" @click="showContact = false">关闭</button>
      </div>
    </div>

    <!-- 标题推荐优化弹窗 -->
    <div v-if="showTitleRecommendModal" class="modal-overlay" @click.self="showTitleRecommendModal = false">
      <div class="modal-content title-modal-content">
        <h3>智能学术标题推荐优化</h3>
        <p>基于您当前输入的关键词及学术热点，推荐以下标准论文题目：</p>
        <div class="title-recommend-list">
          <div 
            v-for="(tItem, idx) in recommendedTitleList" 
            :key="idx" 
            class="title-recommend-item"
            @click="applyRecommendedTitle(tItem)"
          >
            <span class="title-num-badge">0{{ idx + 1 }}</span>
            <span class="title-text">{{ tItem }}</span>
          </div>
        </div>
        <button class="modal-btn secondary" style="margin-top: 1rem;" @click="showTitleRecommendModal = false">关闭</button>
      </div>
    </div>

    <!-- 裂变拦截弹窗 (模块四：裂变机制) -->
    <FissionModal 
      :visible="showFission" 
      :wechat-id="wechatId"
      @unlocked="handleUnlocked"
    />

    <!-- 分享卡片弹窗 (模块二扩展) -->
    <ShareCardModal
      :visible="showShareCard"
      :content="displayResultText"
      :wechat-id="wechatId"
      @close="showShareCard = false"
    />

    <!-- 微信 H5 分享引导浮层 -->
    <div v-if="showShareGuide" class="share-guide-overlay" @click="handleShareClose">
      <div class="share-guide-arrow">↗</div>
      <div class="share-guide-content">
        <p>点击右上角菜单 <strong>•••</strong></p>
        <p>选择 <strong>「分享到朋友圈」</strong></p>
        <p class="share-guide-sub">分享这款好用的 AI 论文写作与学术大纲微应用</p>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue';
import UserTicker from './components/UserTicker.vue';
import FissionModal from './components/FissionModal.vue';
import DemoShowcase from './components/DemoShowcase.vue';
import ShareCardModal from './components/ShareCardModal.vue';
import appConfig from './config.json';
const weixinImg = 'https://ai.wuxian.xyz/assets/weixin.png';
const dingtalkImg = 'https://ai.wuxian.xyz/assets/dingtalk.png';

const appTitle = ref(appConfig.title || '网腾无限AI - 论文写作与学术大纲生成专家');
const wechatId = ref(appConfig.wechatId || 'ai_wuxian_xyz');
const promptTopic = ref(appConfig.promptTopic || '');

// 参考图一：23 种论文/报告类型
const paperTypeList = [
  '毕业论文', '开题报告', '文献综述', '课程论文', '课程作业', 
  '期刊论文', '职称论文', '专升本论文', '科学小论文', '教学设计', 
  '职业规划', '实习报告', '读后感', '心得体会', '写报告', 
  '答辩稿', '调查报告', '论文任务书', '中期报告', '课题论文', 
  '自定义写作', '期刊文章', '专著'
];
const activePaperType = ref('毕业论文');

const destination = ref('基于深度学习的智能网联汽车碰撞预警系统设计与实现');

// 参考图一：学术配置单选项
const degreeList = ['专科', '本科', '研究生', '博士', 'MBA'];
const selectedDegree = ref('本科');

const wordCountList = ['3000', '5000', '8000', '1万', '2万', '3万', '5万', '10万', '更多'];
const selectedWordCount = ref('1万');

const selectedModel = ref('标准模型');

const outlineLevelList = ['二级大纲', '三级大纲'];
const selectedOutline = ref('二级大纲');

const languageList = ['中文', '英语', '日语', '韩语', '俄语', '泰语'];
const selectedLanguage = ref('中文');

const loading = ref(false);
const errorMsg = ref('');
const result = ref('');
const copied = ref(false);
const showSuccessToast = ref(false);
const showFission = ref(false);
const showPrivacy = ref(false);
const showTerms = ref(false);
const showContact = ref(false);
const showShareGuide = ref(false);
const showShareCard = ref(false);

const showTitleRecommendModal = ref(false);
const recommendedTitleList = ref<string[]>([]);

const isStamping = ref(false);
const totalCheckin = ref(parseInt(localStorage.getItem('lw_total_academic_stamp') || '0', 10));
const floatingItems = ref<{ id: number; text: string; x: number; y: number }[]>([]);
let floatingId = 0;

// 学术指标评估列表
const metricsList = [
  { key: 'academicRigor', label: '论述严谨度 (Academic Rigor)' },
  { key: 'literatureDepth', label: '文献丰富度 (Literature Depth)' },
  { key: 'outlineLogic', label: '大纲逻辑度 (Outline Logic)' },
  { key: 'innovationValue', label: '创新价值度 (Innovation Value)' },
  { key: 'formatNorm', label: '格式规范度 (Format Norm)' }
];

const aiScores = ref<Record<string, number> | null>(null);

interface HistoryItem {
  id: string;
  timestamp: string;
  paperType: string;
  destination: string;
  degree: string;
  wordCount: string;
  aiScores: Record<string, number> | null;
  output: string;
}

const historyList = ref<HistoryItem[]>([]);
const showHistory = ref(false);

const loadHistory = () => {
  try {
    const raw = localStorage.getItem('lw_history_records');
    historyList.value = raw ? JSON.parse(raw) : [];
  } catch (e) {
    historyList.value = [];
  }
};

const saveHistory = () => {
  localStorage.setItem('lw_history_records', JSON.stringify(historyList.value));
};

const addHistoryRecord = () => {
  const newItem: HistoryItem = {
    id: Date.now().toString(),
    timestamp: new Date().toLocaleString(),
    paperType: activePaperType.value,
    destination: destination.value,
    degree: selectedDegree.value,
    wordCount: selectedWordCount.value,
    aiScores: aiScores.value ? { ...aiScores.value } : null,
    output: result.value
  };
  historyList.value.unshift(newItem);
  saveHistory();
};

const deleteHistoryRecord = (id: string) => {
  historyList.value = historyList.value.filter(item => item.id !== id);
  saveHistory();
};

const clearAllHistory = () => {
  if (confirm('确认清空所有历史论文大纲报告吗？此操作不可恢复。')) {
    historyList.value = [];
    saveHistory();
  }
};

const selectHistoryItem = (item: HistoryItem) => {
  activePaperType.value = item.paperType;
  destination.value = item.destination;
  selectedDegree.value = item.degree;
  selectedWordCount.value = item.wordCount;
  aiScores.value = item.aiScores ? { ...item.aiScores } : null;
  result.value = item.output;
  showHistory.value = false;
};

const getAverageScore = (item: HistoryItem) => {
  if (!item.aiScores) return '4.0';
  const s = item.aiScores;
  const avg = (s.academicRigor + s.literatureDepth + s.outlineLogic + s.innovationValue + s.formatNorm) / 5;
  return avg.toFixed(1);
};

// 标题推荐优化处理
const handleRecommendTitle = () => {
  const baseKeyword = destination.value.trim() || '智能科技与经济发展';
  recommendedTitleList.value = [
    `基于大数据的${baseKeyword}关键技术演进与实证评估机制`,
    `“双碳”目标下${baseKeyword}的系统构建与演进路径探析`,
    `数字化转型视阈下${baseKeyword}的机理研究与对策分析`
  ];
  showTitleRecommendModal.value = true;
};

const applyRecommendedTitle = (title: string) => {
  destination.value = title;
  showTitleRecommendModal.value = false;
};

// 纯前端利用 Web Audio API 动态合成学术铜铃声效 (正弦波 C5 523Hz -> E5 659Hz -> G5 784Hz -> C6 1046Hz 铜铃分解)
const stampCheckin = () => {
  isStamping.value = true;
  setTimeout(() => {
    isStamping.value = false;
  }, 100);

  // 增加打卡计数
  totalCheckin.value++;
  localStorage.setItem('lw_total_academic_stamp', totalCheckin.value.toString());

  // 漂浮动效 (严格无 Emoji)
  const id = floatingId++;
  const x = (Math.random() - 0.5) * 60;
  const y = -40 - Math.random() * 20;
  floatingItems.value.push({ id, text: '治学严谨学术有成', x, y });
  setTimeout(() => {
    floatingItems.value = floatingItems.value.filter(item => item.id !== id);
  }, 800);

  try {
    const audioCtx = new (window.AudioContext || (window as any).webkitAudioContext)();
    
    // 铜铃分解和弦 (523, 659, 784, 1046)
    const freqs = [523.25, 659.25, 783.99, 1046.50];
    freqs.forEach((freq, index) => {
      const osc = audioCtx.createOscillator();
      const gain = audioCtx.createGain();
      osc.type = 'sine'; // 铜铃纯音
      
      const startTime = audioCtx.currentTime + index * 0.03;
      osc.frequency.setValueAtTime(freq, startTime);

      gain.gain.setValueAtTime(0.3, startTime);
      gain.gain.exponentialRampToValueAtTime(0.005, startTime + 0.12);

      osc.connect(gain);
      gain.connect(audioCtx.destination);
      osc.start(startTime);
      osc.stop(startTime + 0.13);
    });
  } catch (err) {
    // 捕获异常
  }
};

const parseAIScores = (text: string) => {
  const match = text.match(/\[LUNWEN_SCORES\](.*?)\[\/LUNWEN_SCORES\]/);
  if (match) {
    const scoreStr = match[1].trim();
    const scores: Record<string, number> = {};
    scoreStr.split(',').forEach(item => {
      const [key, val] = item.split(':');
      if (key && val) {
        scores[key.trim()] = Math.min(5, Math.max(1, parseInt(val.trim(), 10) || 3));
      }
    });
    return {
      academicRigor: scores.academicRigor || 3,
      literatureDepth: scores.literatureDepth || 3,
      outlineLogic: scores.outlineLogic || 3,
      innovationValue: scores.innovationValue || 3,
      formatNorm: scores.formatNorm || 3
    };
  }
  return null;
};

const cleanResponseText = (text: string) => {
  return text.replace(/\[LUNWEN_SCORES\][\s\S]*?\[\/LUNWEN_SCORES\]/gi, '').trim();
};

const displayResultText = computed(() => {
  return cleanResponseText(result.value);
});

const cleanExcerpt = (text: string) => {
  const cleaned = cleanResponseText(text);
  return cleaned.length > 80 ? cleaned.slice(0, 80) + '...' : cleaned;
};

onMounted(() => {
  loadHistory();
});

const handleShareClose = () => {
  showShareGuide.value = false;
  localStorage.setItem('shared_fission', 'true');
};

const isLimitReached = computed(() => {
  const uses = parseInt(localStorage.getItem('free_uses') || '0', 10);
  const shared = localStorage.getItem('shared_fission') === 'true';
  return uses >= 3 && !shared;
});

const apiEndpoint = import.meta.env.DEV
  ? '/api/local/generate'
  : (import.meta.env.VITE_API_ENDPOINT || 'https://api.wuxian.xyz/api/v1/generate');

const triggerSuccessToast = () => {
  showSuccessToast.value = true;
  setTimeout(() => {
    showSuccessToast.value = false;
  }, 3000);
};

const handleGenerate = async () => {
  if (isLimitReached.value) {
    showFission.value = true;
    return;
  }

  loading.value = true;
  errorMsg.value = '';
  result.value = '';
  aiScores.value = null;

  try {
    const fullPrompt = `【论文类型】：${activePaperType.value}\n【题目及写作要求】：${destination.value}\n【学历层次】：${selectedDegree.value}\n【期望字数】：${selectedWordCount.value}\n【AI模型引擎】：${selectedModel.value}\n【大纲层级】：${selectedOutline.value}\n【撰写语言】：${selectedLanguage.value}\n${promptTopic.value}`;

    const response = await fetch(apiEndpoint, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      credentials: 'include',
      body: JSON.stringify({
        taskType: 'text',
        prompt: fullPrompt,
        style: `${activePaperType.value} (${selectedDegree.value})`
      })
    });

    const data = await response.json();
    if (data.error) {
      errorMsg.value = data.error;
    } else {
      result.value = data.result;
      aiScores.value = parseAIScores(data.result);
      addHistoryRecord();
      triggerSuccessToast();
      
      const currentUses = parseInt(localStorage.getItem('free_uses') || '0', 10);
      localStorage.setItem('free_uses', (currentUses + 1).toString());
    }
  } catch (err: any) {
    errorMsg.value = '请求接口失败，请检查网络或本地代理服务。';
  } finally {
    loading.value = false;
  }
};

const resetReview = () => {
  result.value = '';
  aiScores.value = null;
};

const handleUseSample = (sampleTopic: string, sampleDestination: string) => {
  destination.value = sampleDestination;
  showHistory.value = false;
  window.scrollTo({ top: 0, behavior: 'smooth' });
};

const handleUnlocked = () => {
  showFission.value = false;
  handleGenerate();
};

const copyText = async () => {
  try {
    await navigator.clipboard.writeText(displayResultText.value);
    copied.value = true;
    setTimeout(() => {
      copied.value = false;
    }, 2000);
  } catch (err) {
    errorMsg.value = '复制失败，请手动选择复制。';
  }
};
</script>
