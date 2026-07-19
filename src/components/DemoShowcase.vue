<template>
  <section class="glass-card showcase-container">
    <div class="showcase-header">
      <div class="showcase-title-group">
        <h2 class="showcase-title">论文写作与学术开题案例库 (30 精选样例)</h2>
        <p class="showcase-subtitle">涵盖工学、理学、经济学、管理学、文学、法学等学科的毕业论文、开题报告与期刊综述，点击“一键同款生成”即可即刻核算</p>
      </div>
      <div class="showcase-badge">学术规范 · 免费体验</div>
    </div>

    <!-- 搜索与分类筛选 -->
    <div class="showcase-filter-bar">
      <div class="category-tabs">
        <button 
          v-for="cat in categories" 
          :key="cat"
          class="category-tab"
          :class="{ active: currentCategory === cat }"
          @click="currentCategory = cat"
        >
          {{ cat }}
        </button>
      </div>
      <div class="search-input-wrapper">
        <input 
          v-model="searchQuery"
          type="text"
          placeholder="搜索论文题目、学科专业、研究方法、大纲层级或关键字..."
          class="search-input"
        />
      </div>
    </div>

    <!-- 样例列表格 Grid -->
    <div class="sample-grid">
      <div 
        v-for="sample in paginatedSamples" 
        :key="sample.id" 
        class="sample-card"
      >
        <div class="sample-card-header">
          <span class="topic-category-tag">{{ sample.category }}</span>
          <span class="style-name-tag">{{ sample.degree }} · {{ sample.words }}</span>
        </div>
        <div class="sample-original">
          <span class="sample-label">论文题目：</span>“{{ sample.destination }}”
        </div>
        <div class="sample-rewritten">
          <span class="sample-label">写作大纲概要：</span>{{ sample.core }}
        </div>
        <div class="sample-card-footer">
          <button class="use-sample-btn" @click="$emit('use-sample', sample.topic, sample.destination)">
            一键同款生成
          </button>
        </div>
      </div>
    </div>

    <!-- 空状态提示 -->
    <div v-if="filteredSamples.length === 0" class="empty-showcase">
      未找到匹配的学术论文案例，请尝试切换分类或重置搜索关键词。
    </div>

    <!-- 分页组件 -->
    <div v-if="filteredSamples.length > pageSize" class="pagination-bar">
      <button 
        class="page-btn" 
        :disabled="currentPage === 1"
        @click="currentPage--"
      >
        上一页
      </button>
      <span class="page-info">第 {{ currentPage }} / {{ totalPages }} 页 (共 {{ filteredSamples.length }} 条)</span>
      <button 
        class="page-btn" 
        :disabled="currentPage === totalPages"
        @click="currentPage++"
      >
        下一页
      </button>
    </div>
  </section>
</template>

<script setup lang="ts">
import { ref, computed, watch } from 'vue';

defineEmits<{
  (e: 'use-sample', text: string, destination: string): void;
}>();

const categories = ['全部', '毕业论文', '开题报告', '文献综述', '期刊论文', '答辩稿'];
const currentCategory = ref('全部');
const searchQuery = ref('');
const currentPage = ref(1);
const pageSize = 6;

interface LunwenSample {
  id: number;
  category: string;
  destination: string;
  topic: string;
  degree: string;
  words: string;
  core: string;
}

// 精选 30 条论文写作与学术开题案例
const raw30Samples: LunwenSample[] = [
  {
    id: 1,
    category: '毕业论文',
    destination: '基于深度学习的智能网联汽车碰撞预警系统设计与实现',
    topic: '关注毫米波雷达与视觉融合算法、三维轨迹预测与碰撞时间TTC计算。',
    degree: '本科',
    words: '1万字',
    core: '大纲：绪论（研究背景与意义）、传感器数据融合架构设计、碰撞预警模型构建、仿真测试与实车验证、总结与展望。'
  },
  {
    id: 2,
    category: '开题报告',
    destination: '数字经济对制造企业绿色创新绩效的传导机制研究',
    topic: '关注中介效应模型、面板数据实证分析与环境规制调节作用。',
    degree: '研究生',
    words: '5000字',
    core: '大纲：选题依据与研究创新、文献综述与理论假设、研究设计与计量模型、数据来源与统计描述、预期成果与进度规划。'
  },
  {
    id: 3,
    category: '文献综述',
    destination: '大语言模型在智能教育领中的应用演进与未来趋势综述',
    topic: '关注提示工程、自适应学习路径、知识图谱融合与学术伦理挑战。',
    degree: '研究生',
    words: '8000字',
    core: '大纲：引言、LLM教育应用演进脉络、个性化推荐与答疑核心技术、学术伦理与技术局限分析、未来发展演化路径。'
  },
  {
    id: 4,
    category: '期刊论文',
    destination: '高能密度锂硫电池固态电解质界面膜（SEI）调控策略',
    topic: '关注电化学阻抗谱分析、原位TEM观测与枝晶抑制机制。',
    degree: '博士',
    words: '1.5万字',
    core: '大纲：摘要与Abstract、实验材料与制备、SEI膜形貌与元素表征、电化学性能测试分析、反应机制讨论与结论。'
  },
  {
    id: 5,
    category: '答辩稿',
    destination: '新媒体时代乡村文化旅游品牌传播与营销策略研究答辩演讲稿',
    topic: '关注PPT结构对应、研究背景阐述、核心创新点强调与答辩应对准备。',
    degree: '本科',
    words: '3000字',
    core: '大纲：各位评委老师好（开场致谢）、课题研究缘起与现实意义、核心案例对比分析、策略建议与结论、提问环节致谢。'
  },
  {
    id: 6,
    category: '毕业论文',
    destination: '基于 Vue3 和 Node.js 的高并发微前端电商平台系统设计',
    topic: '关注无缝路由切换、单点登录SSO、Redis缓存高并发与分布式架构。',
    degree: '本科',
    words: '1万字',
    core: '大纲：开发背景与需求分析、系统总体架构设计、微前端子应用隔离实现、高并发缓存与秒杀模块、系统测试与性能优化。'
  },
  {
    id: 7,
    category: '开题报告',
    destination: '乡村振兴战略背景下现代农业园区产业链延伸路径研究',
    topic: '关注三产融合机制、SWOT分析与产业延伸定量评价指标体系。',
    degree: '研究生',
    words: '5000字',
    core: '大纲：立项背景与现实紧迫性、国内外研究现状评估、研究内容与技术路线图、重点难点与创新之处、参考文献目录。'
  },
  {
    id: 8,
    category: '文献综述',
    destination: '近十年国内中小学心理健康教育干预模式对比综述',
    topic: '关注积极心理学视角、家庭学校协同机制与干预效果Meta分析。',
    degree: '本科',
    words: '5000字',
    core: '大纲：引言与检索范围说明、心理健康干预模式分类演变、家校协同干预实证对比、存在问题与改进构想、总结。'
  },
  {
    id: 9,
    category: '期刊论文',
    destination: '企业碳信息披露质量对融资成本的影响实证研究',
    topic: '关注ESG评价体系、双重差分法DID与双向固定效应模型。',
    degree: 'MBA',
    words: '8000字',
    core: '大纲：研究背景与假设提出、研究设计与变量定义、实证检验与结果分析、稳健性检验与异质性分析、政策建议与结论。'
  },
  {
    id: 10,
    category: '答辩稿',
    destination: '智能推荐算法中的个人信息保护法律困境与对策毕业答辩稿',
    topic: '关注法条引用、用户隐私侵权模式阐述与合规立法建言。',
    degree: '本科',
    words: '3000字',
    core: '大纲：演讲开场白、选题立意与法学价值、智能算法隐私侵权表现、法治修补对策阐述、结语与恳请老师批评指正。'
  },
  {
    id: 11,
    category: '毕业论文',
    destination: '基于区块链技术的跨境供应链金融信任机制与风险控制',
    topic: '关注智能合约设计、去中心化身份识别与供应链穿透式监管。',
    degree: '研究生',
    words: '2万字',
    core: '大纲：绪论、跨境供应链金融信用博弈模型、基于区块链的信用穿透系统架构、风险识别与智能控制、系统仿真与结论。'
  },
  {
    id: 12,
    category: '开题报告',
    destination: '多源遥感数据融合在湿地生态植被覆盖度动态监测中的应用',
    topic: '关注Sentinel-2与Landsat数据融合、NDVI时空序列分析。',
    degree: '研究生',
    words: '5000字',
    core: '大纲：研究依据与湿地生态监测意义、遥感数据融合方法综述、技术路线与数据处理流程、预期成果与进度安排。'
  },
  {
    id: 13,
    category: '文献综述',
    destination: '西方现代主义文学中“疏离感”主题的演变历程综述',
    topic: '关注卡夫卡、加缪等作家作品比较、存在主义哲学背景。',
    degree: '本科',
    words: '6000字',
    core: '大纲：前言、工业革命与异化理论根源、现代主义小说中的疏离感文本表征、文学批评视角对比、总结与余论。'
  },
  {
    id: 14,
    category: '期刊论文',
    destination: '柔性可穿戴压力传感器高灵敏度结构工程与微结构设计',
    topic: '关注金字塔微结构、PDMS柔性衬底与微弱脉搏信号检测。',
    degree: '博士',
    words: '1.2万字',
    core: '大纲：Introduction、Experimental Section、Microstructure Modeling、Results and Discussion、Conclusion。'
  },
  {
    id: 15,
    category: '答辩稿',
    destination: '基于卷积神经网络的医学CT图像肺结节自动检测系统答辩稿',
    topic: '关注数据增强、U-Net分割网络精度提升与临床辅助价值。',
    degree: '研究生',
    words: '4000字',
    core: '大纲：开场致辞、课题背景与医学痛点、算法创新与网络结构设计、DICOM数据集实验对比、总结与致谢。'
  },
  {
    id: 16,
    category: '毕业论文',
    destination: '跨文化交际视角下中国国产电影英译字幕的文化归化与异化',
    topic: '关注翻译目的论、文化负载词英译与受众心理接受度。',
    degree: '本科',
    words: '8000字',
    core: '大纲：引言、翻译目的论与字幕翻译特点、文化负载词分类及翻译案例分析、归化与异化策略选择平衡、结论。'
  },
  {
    id: 17,
    category: '开题报告',
    destination: '城市地下管网综合走廊智能化运维平台建设可行性研究',
    topic: '关注BIM+GIS技术融合、数字孪生模型构建与安全预警机制。',
    degree: 'MBA',
    words: '5000字',
    core: '大纲：项目背景与研究必要性、技术可行性与经济效益评估、数字孪生运维平台功能设计、工作计划与阶段目标。'
  },
  {
    id: 18,
    category: '文献综述',
    destination: '行为金融学视角下投资者情绪对股票市场波动影响研究综述',
    topic: '关注羊群效应、过度自信代理变量测量与股市异常现象解释。',
    degree: '研究生',
    words: '8000字',
    core: '大纲：引言、投资者情绪度量指标分类综述、情绪对资产定价传导机制、国内股市实证对比、研究不足与展望。'
  },
  {
    id: 19,
    category: '期刊论文',
    destination: '基于强化学习的高效能无刷直流电机转速控制算法研究',
    topic: '关注DDPG深度确定性策略梯度、负载突变响应速度与超调量优化。',
    degree: '研究生',
    words: '1万字',
    core: '大纲：引言、BLDCM电机数学模型建立、强化学习控制器设计、Matlab/Simulink硬件在环实验、结果讨论与结论。'
  },
  {
    id: 20,
    category: '答辩稿',
    destination: '品牌社交媒体营销对消费者购买意愿的影响研究答辩稿',
    topic: '关注SOR模型、社交互动与品牌认同中介变量解读。',
    degree: '本科',
    words: '3000字',
    core: '大纲：尊敬的各位老师好、研究背景与调查问卷发放说明、结构方程模型分析结论、营销策略建议、答辩陈述完毕。'
  },
  {
    id: 21,
    category: '毕业论文',
    destination: '基于大数据的电商平台用户流失预测与精准挽留策略',
    topic: '关注XGBoost分类算法、RFM模型特征工程与A/B测试验证。',
    degree: '本科',
    words: '1万字',
    core: '大纲：绪论、电商数据预处理与特征工程、XGBoost预测模型搭建、召回与挽留策略设计、模型评估与结论。'
  },
  {
    id: 22,
    category: '开题报告',
    destination: '面向智能网联环境的信号交叉口多车协调控制算法',
    topic: '关注V2X车联网通信、延迟时间最小化与车队车速平滑算法。',
    degree: '研究生',
    words: '5000字',
    core: '大纲：选题依据与交通拥堵背景、国内外车联网控制综述、算法流程图与求解思路、时间节点与预估难题。'
  },
  {
    id: 23,
    category: '文献综述',
    destination: '微塑料在海洋生态系统中的分布、迁移及生物毒性效应综述',
    topic: '关注海洋食物链富集、细胞毒性机制与全球治理公约。',
    degree: '研究生',
    words: '7000字',
    core: '大纲：引言、微塑料来源与环境分布现状、海洋生物吸收与食物链迁移、分子与细胞毒理机制、国际管控政策综述。'
  },
  {
    id: 24,
    category: '期刊论文',
    destination: '双碳目标下新型电力系统储能配置优化与经济性评价',
    topic: '关注风光发电随机性、混合储能电池配置与NPV净现值计算。',
    degree: '研究生',
    words: '9000字',
    core: '大纲：摘要、新型电力系统出力特性分析、储能容量双层优化配置模型、算例分析与经济性对比、结论与建议。'
  },
  {
    id: 25,
    category: '答辩稿',
    destination: '古建筑木结构防腐与抗震加固保护工程设计答辩稿',
    topic: '关注榫卯结构受力分析、碳纤维布加固效果与文物保护原则。',
    degree: '本科',
    words: '3500字',
    core: '大纲：开场致意、古建筑病害调查汇报、加固方案设计亮点、有限元数值模拟验证、总结致谢。'
  },
  {
    id: 26,
    category: '毕业论文',
    destination: '人工智能时代劳动法律关系的界定与劳动者权益保障',
    topic: '关注平台算法控制、零工经济劳动关系判定与工伤保险制度。',
    degree: '本科',
    words: '1.2万字',
    core: '大纲：引言、AI零工就业劳动关系认定困境、算法管理对传统劳动法的冲击、国外立法经验借鉴、我国法律应对方案。'
  },
  {
    id: 27,
    category: '开题报告',
    destination: '基于知识图谱的高校课程个性化推荐系统设计与实现',
    topic: '关注Neo4j图数据库、先修课程依赖图构建与协同过滤结合。',
    degree: '本科',
    words: '4500字',
    core: '大纲：课题背景与选课痛点、知识图谱与推荐算法技术方案、系统模块框架、开发计划与参考文献。'
  },
  {
    id: 28,
    category: '文献综述',
    destination: '心理韧性（Resilience）在青少年应激事件处理中的中介效应综述',
    topic: '关注量表测量工具、家庭教养方式与神经机制研究。',
    degree: '研究生',
    words: '6000字',
    core: '大纲：引言、心理韧性概念与理论模型演变、应激事件防护因子分析、测评工具对比、未来研究突破口。'
  },
  {
    id: 29,
    category: '期刊论文',
    destination: '基于无人机高光谱遥感的农作物病虫害早期识别方法',
    topic: '关注高光谱反射率曲线、支持向量机SVM分类与空间差值。',
    degree: '研究生',
    words: '8500字',
    core: '大纲：Abstract、无人机遥感数据采集、波段特征提取算法、病虫害反演模型构建、分类精度验证与讨论。'
  },
  {
    id: 30,
    category: '答辩稿',
    destination: '我国高新技术企业研发费用加计扣除税务筹划与合规管理答辩稿',
    topic: '关注税收优惠政策解读、研发费用归集合规与风险应对。',
    degree: 'MBA',
    words: '3000字',
    core: '大纲：评委老师好、选题背景与企业税负痛点、研发费用合规归集方案、风险防范与财务效益、恳请老师点评。'
  }
];

const samples = ref<LunwenSample[]>(raw30Samples);

const filteredSamples = computed(() => {
  return samples.value.filter(s => {
    const matchCat = currentCategory.value === '全部' || s.category === currentCategory.value;
    const matchQuery = !searchQuery.value.trim() || 
      s.topic.includes(searchQuery.value) || 
      s.destination.includes(searchQuery.value) ||
      s.degree.includes(searchQuery.value) || 
      s.core.includes(searchQuery.value);
    return matchCat && matchQuery;
  });
});

const totalPages = computed(() => Math.ceil(filteredSamples.value.length / pageSize) || 1);

const paginatedSamples = computed(() => {
  const start = (currentPage.value - 1) * pageSize;
  return filteredSamples.value.slice(start, start + pageSize);
});

watch([currentCategory, searchQuery], () => {
  currentPage.value = 1;
});
</script>
