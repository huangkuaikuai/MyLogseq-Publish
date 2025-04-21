- 本文探讨了AI驱动的中后台前端研发实践，涵盖设计出码、接口定义转换、代码拟合、自动化测试等多个环节，通过具体案例展示了AI技术如何优化研发流程并提升效率。特别是在UI代码编写和接口联调阶段，并提出了设计出码（Design to Code）、接口定义到数据模型转换、代码拟合与调整、自动化测试回归等解决方案。同时，介绍了基于大语言模型的私有组件支持、RAG方案以及AI辅助的Code Review工具。最后，文章总结了试点结果，展示了AI在中后台场景下的应用效果，并展望了未来AI在研发流程中的深度整合与发展方向。
- # 01 背景
	- 从Anthropic今年2月发布的AI经济分析报告中可以看到，现阶段对社会各行各业来说，AI影响最大的仍然是计算软件和编程相关领域。在Claude中的所有对话中占比达到了37%，而在编程领域中，前端结合AI的空间是非常大的，因为业务前端是以一个个页面的形态交付，每个APP之间比较独立，不像后端应用中有着复杂的依赖的调用关系，更加适合AI去创作和理解。
	- ![图片](https://mmbiz.qpic.cn/mmbiz_png/33P2FdAnjuibtTBDB96EIdhT3fe5R4wibCMiclX4go6ITeHmxYps4oSqJxWcG3V85nvyERLWUHK2MXevA7Krjg72Q/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
	- 对于研发提效整体的思路是：在研发流程的各个阶段将问题进行拆解，在不同的流程中融入AI Agent能力进行研发节点的提效。在实践的过程中，沉淀产品化的能力，并最终落地形成AI驱动中后台研发解决方案。
	- ## 效率问题分析
		- 一个产品需求迭代从流程上我们一般从大的阶段上分为以下几个阶段：
		- 需求评估 -> 研发 -> 联调 -> 测试 -> 交付。
		- 从研发开始的流程再往下细分，不同的BU，不同的业务线由于使用的技术栈会存在部分的差异，以岗位的视角来看，一般可前端和后端可以拆分成以下几个流程：
		- 后端研发流程： PRD -> UI -> 技术评审/接口设计 -> 变更 -> 代码开发 -> 网关注册 -> 联调修改 -> 线上发布
		- **前端研发流程：PRD -> UI -> 技术评审/接口设计 -> 创建应用/变更 -> 开发 -> 联调 -> 线上发布**
		- 我们统计了团队内部的同学在一线业务需求研发的过程中，研发流程各个阶段的耗时比例，统计的方式主要是通过一线开发同学的主观反馈，因为一个需求研发过程并不是连贯的，大部分的同学手里同时都在做多个需求，并且也会由于各种客观原因，比如需求发生调整，业务以来信息还没有准备完成，人员抽调等等，很难进行精确的时效统计。
		- 研发流程的耗时分布统计：
			- ![图片](https://mmbiz.qpic.cn/mmbiz_png/33P2FdAnjuibtTBDB96EIdhT3fe5R4wibCbYDGH3TH2XibpQicsowjgG4fReibOric6sELklj67lOkCDDF6TymHnFCNQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
		- （注：联调阶段包含联调的接口业务逻辑编写和调试，代码编写特指设计稿到前端交互代码的编码阶段）
		- 从上述的阶段分析来看，**对于研发效率主要能提升的节点在于UI代码编写和接口连调阶段。**
		- 到具体的反馈来看，联调的耗时问题的集中在：
		- 需求频繁变动：需求变更频繁，前后端需要快速响应调整，这对双方的沟通效率和代码灵活性提出了更高要求。
		- 文档滞后与不准确：接口文档常常无法及时更新或描述不够准确，导致前后端开发人员依据过时或错误的信息进行开发。
		- 接下来主要围绕这2个场景介绍一下我们在提效过程中的一些方案设计推导和实践。
- # 02 实践推导
	- ## 设计出码
	  collapsed:: true
		- 在设计出码的这个链路中，已经有很多的同类型的产品，有的产品会选择DSL的转化路线，比如Figma/mgdone的砖码插件，支付宝的WeaveFox，还有大部分低代码平台，虽然通过DSL中间层实现代码生成，相较于普通AI直出代码，优势在于程序化解析保障稳定性（防错机制）和统一DSL支撑跨语言协同。
		- 我们选择了AI直出代码的方案，主要考虑到几下几点：
			- DSL大部分是各个平台私有化的定制，缺少统一的标准化，对于模型学习的语料不足，或者说需要进行一定的预训练，而前端代码，无论是React/Vue都有海量的公共学习语料，随着模型的学习数据和理解力的不断提升，完全能达到初/中级前端程序员的编码能力。
			- 面向B端以中后台为主的页面和面向C端以导购营销为主的页面存在较大的差别，C端页面会存在各种营销氛围的叠加，一个商品坑位存在好几层的图层堆叠，使用DSL转换辅助可以保障AI出码的稳定性，但是**中后台的场景更偏功能性，每个区块分布都比较独立，使用最新的Anthropic Claude Sonnet3.7模型出码已经能以较高的还原度满足开发诉求**，以下是一个中后台的例子：
		- 设计稿：
			- ![图片](https://mmbiz.qpic.cn/mmbiz_png/33P2FdAnjuibtTBDB96EIdhT3fe5R4wibCmNVm3XYdK0c6uZcBHMPI5Vtw7OY1X1fAjEx98hLEsVicCZWqv6Sx0PA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
		- AI出码：
			- ![图片](https://mmbiz.qpic.cn/mmbiz_png/33P2FdAnjuibtTBDB96EIdhT3fe5R4wibC5hSa5h7hqqdicL2L2r1icDtINV7FuGNPjLMSKTOdEwpjRibbJK3vFQovg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
	- ## 模型选择和提示词
	  collapsed:: true
		- ### 1 Claude 3.7 Sonnet
			- 在模型的选择方面，Claude 3.7 sonnet V2在代码方面的能力毋庸置疑，已经甩开了与OpenAI主流模型的差距,新版本在主动编码和工具使用方面有明显进步。
			- 在编码测试中，它将SWE-bench Verified的表现从33.4%提高到了49.0%，超过了所有公开的模型，不仅包括OpenAI o1-preview这样的推理模型，还有专为主动编码设计的系统。
			- **在我们内部的测试中，面向前端开发，无论是面向UI出码还是后续的代码拟合过程中，****Claude 3.7 sonnet 在横向的多种模型对比下均表现出最佳的代码生成能力，问题解决能力，架构设计能力**。以下是Anthropic官方在2025年2月25日发布的最新评分：
			- ![图片](https://mmbiz.qpic.cn/mmbiz_png/33P2FdAnjuibtTBDB96EIdhT3fe5R4wibC8qJ7vVuibrzOh05biaH5QR0CtrLsgUQvoz1HPtSohF2ZMhZb2XAGI8Og/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
			- ![图片](https://mmbiz.qpic.cn/mmbiz_png/33P2FdAnjuibtTBDB96EIdhT3fe5R4wibCNOQKOIxzFCibxyICmJWgWor4nYiaWH9vhwqN8vZiaiaSx4lCcQckV7RqibA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
			- ![image.png](../assets/image_1745245049212_0.png)
		- ### 2 提示词编写
		  collapsed:: true
			- 关于D2C出码提示词的编写，业内有很多的参考，比如：
			- cline提示词：https://github.com/cline/cline/blob/main/src/core/prompts/system.ts
			- bolt.new提示词：https://github.com/stackblitz/bolt.new/blob/main/app/lib/.server/llm/prompts.ts
			- 整体的思路都是比较类似的，大致的范式就是：角定定义(role)，系统约束(constraints)，还有具体的示例（few-shot）。
			- 角定定义和技能相关上的描述基本比较通用，比如“你的知识覆盖了各种编程语言、框架和最佳实践，特别注重React和现代Web开发。”，“具有React组件和Hooks的深度开发经验。”，“能够熟练的使用 Fusion(@alifd/next) 组件库进行页面的还原。”
			- 系统约束由于不同业务场景的不同需要，一般是由业务团队自己进行定制，
			- 明确性：react组件代码块仅支持一个文件，没有文件系统。用户不会为不同文件编写多个代码块，也不会在多个文件中编写代码。用户的习惯总是内联所有代码。
			- 约束性：所有的时间格式处理请都使用原生的 js 代码实现，不要使用任何时间处理库。
			- 场景化：不会为组件或库使用动态导入或懒加载。例如，`const Confetti = dynamic(...)`是不允许的。请使用`import Confetti from 'react-confetti'`。
			- 也可以包含一些通用的限制，例如：
				- 1. 避免使用 iframe、视频或其他媒体，因为它们不会在预览中正确渲染。
				- 2. 不会输出`<svg>`图标。总是使用 `@alifd/next` 库中的Icon的图标。
			- prompt示例：
				- ```apl
				  <role>
				    你是一个高级前端开发工程师。基于用户提供的组件描述，请生成一个React Fusion组件代码块。请使用中文回答。
				  </role>
				  
				  <skills>
				    1. 你能够熟练的使用 Fusion(@alifd/next) 组件库进行页面的还原。 
				    2. 你能够熟练的使用 bizcharts
				    图表库进行图表的可视化展示。图表库的导入方式类似`import {AreaChart} from 'bizcharts'`;
				   3. 注意field 是要用Field.useField()
				  </skills>
				  
				  <constraints>
				  使用```tsx 语法来返回 React 代码块。
				  
				  <engineering-constraints>
				    1. React组件代码块仅支持一个文件，没有文件系统。用户不会为不同文件编写多个代码块，也不会在多个文件中编写代码。用户的习惯总是内联所有代码。
				    2. 必须导出一个名为"Component"的函数作为默认导出。 
				    3. 你总是需要返回完整的代码片段，可以直接复制并粘贴到项目工程中执行。不要包含用户补充的注释。
				    4. 代码返回格式需要参考给出的示例代码 
				    ......
				    10. tsx block请务必在第一个返回，后面讲思考过程和解释。
				    11. 请注意UI的布局、颜色、主要按钮等信息，保证和图像中的结构和布局一致。
				    12. 按照 <data-define></data-define>中的数据、hooks定义构建符合字段含义的数据。
				  </engineering-constraints>
				  
				  <attention>
				  1. form用法中应该用Field.useField() 而不是Form.useForm，更要注意Rol Col用法和props
				  2. pay attention on fusion components. 如果不确定有没有对应组件请用div实现
				  3. 请注意严格按照图片中的逻辑、描述进行还原
				  ......
				  </attention>
				  
				  <style-constraints>
				    1. 总是尝试使用 @alifd/next 库，在 @alifd/next 不满足的情况下才通过 div 和 style 属性生成。
				    2. 必须生成响应式设计，生成的代码移动端优先。 
				  ......
				  </style-constraints>
				  
				  </constraints>
				  
				  <good-examples>
				  ......
				  </good-examples>
				  
				  <bad-examples>
				  ......
				  </bad-examples>
				  ```
		- ### 3 私有化组件支持
			- 对于AI出码来说，除了设计稿的准确还原以外，另一部分很重要的是如何把团队的私有的物料组件结合到生成的代码中。
			- 使用业务私有组件的主要原因包括：
				- **设计资产复用**：将团队沉淀的业务组件（如审批流表单、数据看板卡片）转化为AI可识别的设计资产，避免重复造轮子
				- **代码规范统一：**通过私有组件约束代码生成边界，保证AI输出符合企业级代码规范（如数据校验规则、埋点标准）
			- 在AI出码的流程实现融入私有化组件的大致流程如下：
				- 1. 将设计元素与私有组件库特征进行向量化匹配
				- 2. 动态注入组件使用规范、业务逻辑约束等上下文
				- 3. 生成符合企业标准的定制化代码
				- ![图片](https://mmbiz.qpic.cn/mmbiz_png/33P2FdAnjuicFRK8Q7SE9DN5cYfB1P31q5x4N9yV5TtibI91VhdpiaH9u6phhIoJJBLDypfbHjygvMnFvc8icnz3NA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
		- ### 4 组件文档的生成
			- 目前的大语言模型AI的能力，尤其是近期不断更新的reasoning推理型大模型，将私有组件的源代码转换成标准化的组件文档非常的轻松，仅需要编写一段清晰的prompt并且给几个具体的example案例，模型就可以生成非常高标准高质量的markdown格式组件文档。
			- 关注提示词编写的注意事项在上文D2C提示词编写已经说明过了，这里直接给出具体的prompt案例：
			  collapsed:: true
				- ```apl
				  <role>
				    您是一个专注于为前端组件生成清晰结构化文档的文档助手。根据用户提供的 React 组件代码，您的任务是创建与示例文档 `example.md` 格式和风格一致的规范化文档片段。
				  </role>
				  
				  <skills>
				    1. 能有效解析和理解 React 组件代码
				    2. 擅长将代码逻辑、结构和功能转化为精准简明的文档
				    3. 熟悉 @ali/homepage-card 和 bizcharts 库的细节，能在文档中准确描述其用法
				  </skills>
				  
				  <output-constraints>
				    1. 输出必须采用 Markdown 格式
				    2. 输出内容仅包含组件说明，不包含任何礼貌性冗余表述
				    3. 文档应包含组件描述、属性类型、默认值和用法示例
				    4. 注意识别接口中实际未使用但被强制要求填写的字段，需在文档中明确标注
				    5. 生成使用示例时，必须包含原始代码接口定义中标记为必填的参数（即使未实际使用）
				    6. 严格遵循 `example.md` 的样式和格式规范
				    7. 确保文档专业准确，覆盖边界用例和典型场景
				  </output-constraints>
				  
				  <engineering-constraints>
				    1. 分析 React 组件代码以提取必要的文档信息
				    2. 重点将代码逻辑转换为清晰的文档结构（"描述"/"属性"/"示例用法"/"注意事项"）
				    3. 避免技术术语，使用简洁易懂的语言适应广泛读者群体
				  </engineering-constraints>
				  ```
			- 以下是一个完整组件文档的示例
			  collapsed:: true
				- ```apl
				  组件名称和组件的使用场景
				  组件名称: AnalysisText
				  使用场景:
				  AnalysisText 是一个用于展示智能分析报告的卡片组件。它通常用于物流解决方案的首页，展示一些分析数据或报告内容。该组件可以自定义标题、内容以及样式类名，适合用于需要展示富文本内容的场景。
				  
				  该组件的props说明
				  Prop Name	Type	Default Value	Description
				  data	string	""	需要展示的分析报告内容，支持HTML格式。
				  title	ReactNode	<div className="analysis-text-title"><img width={20} src="..."/>智能分析报告</div>	自定义标题，如果不传入，则会使用默认的标题（包含图标和“智能分析报告”文字）。
				  className	string	""	可选的自定义样式类名，用于覆盖默认样式。
				  
				  该组件的使用示例
				  import React from 'react';
				  import { AnalysisText } from '@ali/homepage-card';
				  
				  const App = () => {
				    const reportData = `
				      <p>这是一份智能分析报告的内容。</p>
				      <ul>
				        <li>分析点1：数据趋势良好</li>
				        <li>分析点2：存在部分异常</li>
				      </ul>
				    `;
				  
				    return (
				      <div>
				        {/* 使用默认标题 */}
				        <AnalysisText data={reportData} />
				  
				        {/* 自定义标题 */}
				        <AnalysisText 
				          data={reportData} 
				          title={<h3>自定义分析报告</h3>} 
				          className="custom-analysis-text" 
				        />
				      </div>
				    );
				  };
				  
				  export default App;
				  
				  注意事项
				  1. HTML内容安全性:
				  data 属性支持HTML格式的内容，但由于使用了 dangerouslySetInnerHTML，需要注意传入的内容是否安全，避免XSS攻击。
				  2. 默认标题:
				  如果不传入 title，组件会使用默认的标题（包含图标和“智能分析报告”文字）。如果需要完全自定义标题，可以通过 title 属性传入自定义的React节点。
				  3. 样式覆盖:
				  可以通过 className 属性传入自定义的样式类名，覆盖默认的样式。建议在项目中使用CSS模块或全局样式来管理组件的样式。
				  ```
	- ## RAG的方案
	  collapsed:: true
		- 关于RAG相关的基础背景知识在这里就不详细展开了，外网上可以搜到大量相关的介绍的文章。在使用RAG方案使用私有组件进行出码分为索引(Index)部分和查询(Query)2个阶段：
		- **在索引阶段：**
			- 知识文档的准备：收集和准备组件的文档，这里的文档可以是上文中通过AI生成的文档，也可以是类似Fusion Design之类的一些外网知识信息偏少的公共组件文档。
			- 文本块拆分：对于私有组件而言，一般来说一个组件就是一个md文档，不需要在额外进行拆分。
			- 嵌入模型：通过LamaIndex等服务将文本块转换为向量表示并存储在向量数据库中。
		- **在查询阶段:**
			- 接收查询请求：在发起模型请求阶段判断当前的出码流程是否需要进行私有组件的召回。
			- 查询处理并检索：按设计稿进行拆分准备检索私有组件文档，如果图像识别不够准确的话可以配以文字的描述
			- 生成回答：根据检索到的文档信息中的组件API和使用说明生成回答（代码）。
		- RAG整体流程示意图
			- ![图片](https://mmbiz.qpic.cn/mmbiz_png/33P2FdAnjuicFRK8Q7SE9DN5cYfB1P31qHTEGFqYAmsGjsiaM9GE3xr9c3enhicqR6sLDDqQFTMBAUsr6nkwmXXnQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
		- 实现RAG也有比较多的方式，主要常见的包括以下几种：
		- **通过LlamaIndex 类的RAG框架从0自己搭建**
		  collapsed:: true
			- LlamaIndexTS 是一个面向 TypeScript/JavaScript 生态的检索增强生成（RAG）框架，专为构建私有化知识索引与智能查询系统设计。其核心目标是通过高效的索引结构，将企业私有数据（如组件库、业务文档、设计规范）与大型语言模型（LLM）结合，实现精准的语义检索与上下文增强生成。
			- 知识库构建示例代码
				- ```dart
				  
				  // 加载私有组件文档
				  const componentDocs = await new ComponentDocLoader({
				    repoPath: './src/components',
				    metadataExtractor: (code) => ({
				      props: extractComponentProps(code),
				      usage: extractUsageExamples(code)
				    })
				  }).load();
				  
				  // 创建专用索引
				  const componentIndex = await VectorStoreIndex.fromDocuments(componentDocs, {
				    embedModel: new ComponentEmbeddingModel()
				  });
				  ```
			- 检索示例代码：
				- ```xml
				  const query = "需要一个带校验功能的表单输入框";
				  const results = await componentIndex.asRetriever().retrieve({
				    query,
				    filters: {
				      componentType: 'FormInput',
				      version: '>=2.3.0'
				    }
				  });
				  
				  // 生成代码上下文
				  const context = results.map(r => r.node.getContent());
				  const prompt = buildCodeGenPrompt(query, context);
				  const code = await llm.generate(prompt);
				  ```
			- 自建RAG服务的场景主要适合对知识库的分拆算法，召回逻辑自定义要求比较高的，并且本身有一定的基础研发能力的团队，涉及到的服务能力，向量数据库的维护，尤其是AI基建越来越完善的当下，**一般来说不是特别推荐**。
		- **通过Dify等的AI agent平台搭建流程**
			- 除了通过LamaIndex，LangChain等开发框架进行私有化部署以外，也可以集成化的AI服务平台，从知识库的管理，Prompt的调优，Agent的流程设计实现都可以一站式的完成。
			- 以集团的AI studio为例，首先在Workspace中先创建一个知识库
			  collapsed:: true
				- ![图片](https://mmbiz.qpic.cn/mmbiz_png/33P2FdAnjuicFRK8Q7SE9DN5cYfB1P31qgtDsmAtx8RoQrXt8ErebmJibicYKPZ6sRtK1RoVt8SMgoxx2VY4Gsf8w/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
			- 支持通过语雀文档或者是本地pdf，markdown等常用格式进行知识库内容的上传
			  collapsed:: true
				- ![图片](https://mmbiz.qpic.cn/mmbiz_png/33P2FdAnjuicFRK8Q7SE9DN5cYfB1P31qr5AHVOicABnE4UvrU06ibV54XDAQlrDDg1pL9RQlCLX5yibKSZUWc6H4w/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
			- 创建一个流程编排Agent,将输入的图片，预制的prompt连接到对应的知识库上，完成整个流程的串联。最后根据这个agent流程在workspace中进行测试和调优，就完成了整个服务的搭建，非常的便捷。
			  collapsed:: true
				- ![图片](https://mmbiz.qpic.cn/mmbiz_png/33P2FdAnjuicFRK8Q7SE9DN5cYfB1P31qzjFIFCFkmevebZ20Y8wDy1lLicKrR5tic2ds2NtIEQkWmRHfj2ibg7OHA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
			- RAG的方案最大的问题在于召回的匹配度，由于输入的信息是图片，大模型根据图像识别的理解需要能够准确的理解需求，并根据语意相似度匹配上对应的私有化组件，这个会存在一定的召回失败的概率。
			- **所以在文档上需要有详细的应用场景和对应的案例，或者是通过类似CopyCoder的方案在前置进行一次图片转文字Prompt的解析，帮助更精准的召回。**
			- **在私有化组件数量不大的情况下，不考虑输入token的成本，除了rag的方案，也可以直接将所有的私有化组件文档通过js脚本进行组合成一个大的语料集合输入给生码的prompt中**。现在的大模型对于token的输入限制已经达到了百万级别，对于私有组件不多场景，少了RAG的召回步骤，应用效果是比较好的。
			- 虽然RAG是比较通用的解决方案，但是由于大部分的用户缺乏专业性，会导致在切片和检索的时候没有办法进行非常精准的匹配导致效果不佳。当然本身RAG的方案也在不断的进步，除了传统的文字embeding模式，现在也有类似Graph RAG，DeepSearcher等新的RAG架构，不断的能提升召回的准确率。
	- ## 接口定义到数据模型
	  collapsed:: true
		- 从前后端连调的视角最核心需要解决的问题是定义清楚接口的交付字段，通常来说后端会编写一份连调接口文档，然后根据这份文档约定进行前后端的业务代码编写。理想是美好的，但是在实际的研发过程中会有各种协同上的问题，如：
			- **文档信息不完备，**接口相关的内容信息存在缺失。例如：缺少接口返回response对象最外层Wrapper的结构，导致前端调用出现空指针。
			- **接口定义频繁的进行变更，**后端在技术方案设计到最终实现的过程中，经常会出现一些内部逻辑或者细节的调整，有可能是出现技术方案上没有考虑到的情况，也有可能是产品需求发生变更。有时是在连调的过程中，有时甚至有的时候是在发布前的CodeReview阶段，不仅存在安全隐患，也导致前端对接的成本随之上升。
			- **交付方式不规范**，由于人员的流动等影响，部分业务外包开发缺少在协同过程中的一些规范，经常会有直接把接口信息扔到钉钉聊天中就觉得完成交付，可能是出于觉得编写文档这个过程过于麻烦。
			- ![图片](https://mmbiz.qpic.cn/mmbiz_png/33P2FdAnjuicFRK8Q7SE9DN5cYfB1P31qEzGmvDm19jdIqtmnleVmmCCKKNnHPfqOMic0MxK61AvDaWddLScWoVw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
		- 我们希望借助AI的能力，把接口定义到前端代码生成的过程标准化，对于后端，不需要花时间和精力去编写/维护接口文档，对于前端，在获取到接口定义的时候，自动转换成实体模型及相关请求hooks，减少时间去编写重复性的代码。
		- 整体的技术方案思路大致如下：
			- 通过不同途径（技术文档/PRD/mock平台）的接口定义的语料输入，经由带CoT的推理模型进行思考解析出标准化的OpenAPI schema协议，作为前后端对接的凭证。
			- OpenAPI schema 3.0 规范可以参考：https://spec.openapis.org/oas/v3.0.3.html
			- 根据得到的接口OpenAPI schema，生成对应的接口相关前端代码，model,service,hooks等。
			- 如果需要，也可以通过工程化的链路，推送到对应的网关进行接口的注册（Mtop，IDD网关）。
		- 链路框架如下图所示：
			- ![图片](https://mmbiz.qpic.cn/mmbiz_png/33P2FdAnjuicFRK8Q7SE9DN5cYfB1P31qKp8eicKF9ib9rVmvZqCqDP6hib0OKyRSiaJweibn5pkmHen8blBejeWroCQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
		- ### 获取OpenAPI 3.0 Schema
			- 将接口定义的转换成Open API Schema的语料有很多种，目前我们常见支持的包括以下几种方式：
			- #### 1. 通过Java Interface或接口文档获取接口定义
			  collapsed:: true
				- 服务端对接前端的接口如果是http的，那么一般是在Spring MVP的Controller层。如果是通过MTOP等网关形式提供的,那么接口的定义一般是在client或者api包下的Facade Service中。
				- 所以只要可以拿到后端应用中的controller或者是interface的代码，那么就可以通过让模型去理解接口的字段定义并转换成OpenAPI schema。
				- 目前我们提供了一个管理页面，服务端开发可以在管理页面上粘贴本次迭代变更相关的接口代码或者是技术文档，只要能包含全量的信息即可，存在部分的冗余也没有关系。（这一步后续也可以集成到git webhooks或者终端的指令上，智能去分析仓库中相关涉及这次变更的代码和实体模型）。
				- 如图是一个二方HSF的Java Interface的源码
					- ![图片](https://mmbiz.qpic.cn/mmbiz_png/33P2FdAnjuicFRK8Q7SE9DN5cYfB1P31qczngtJaClHnskK4aHzDQJZ8yUbhpSicw3ibaB6KQmFWiahOA61BXXMQTw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
				- 提交后由CoT推理大模型进行分析，提取关键的函数名和签名参数信息
					- ![图片](https://mmbiz.qpic.cn/mmbiz_png/33P2FdAnjuicFRK8Q7SE9DN5cYfB1P31qX2dQiayIhdiaLwmKAT5twdGS4kwLqgOYw331DUibLmX7RQqSWnhz3Ixyw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
					- ![图片](https://mmbiz.qpic.cn/mmbiz_png/33P2FdAnjuicFRK8Q7SE9DN5cYfB1P31qFl2gHRI1Qw6AyHWUPf2mEEQKuvwcOtKbyzxuBBwXuo7WcxrQzkxnRA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
				- 最后以OpenAPI Schema的格式进行输出
					- ![图片](https://mmbiz.qpic.cn/mmbiz_png/33P2FdAnjuicFRK8Q7SE9DN5cYfB1P31qVYzZgbWWPKP5LqKic5ict37hYd6ibS6Uew62h9KP6C5ELqxtdbibvZT2TQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
				- 这种方式的普适性比较广，但是准确率相对偏低一点，一方面是目前的大模型无法避免的幻觉问题，另一方式则是部分的手动复制的时候可能会遗漏部分的必要信息，可以在左边的OpenAPI的可视化编辑器内进行二次的修改。
			- #### 2. 通过Swagger插件获取接口定义
				- 第二种方式是通过安装Swagger插件，在项目中引入springfox-swagger2的依赖，并初始化配置类，具体的接入方式和demo可以参考SpringFox的官方站点介绍：https://springfox.github.io/springfox/
				- 在maven中引入的配置包括：
				  collapsed:: true
					- ```xml
					  <dependency>
					      <groupId>io.springfox</groupId>
					      <artifactId>springfox-swagger2</artifactId>
					      <version>2.8.0</version>
					  </dependency>
					  <dependency>
					      <groupId>io.springfox</groupId>
					      <artifactId>springfox-swagger-ui</artifactId>
					      <version>2.8.0</version>
					  </dependency>
					  <dependency>
					      <groupId>com.github.xiaoymin</groupId>
					      <artifactId>swagger-bootstrap-ui</artifactId>
					      <version>1.9.6</version>
					  </dependency>
					  ```
			- 完成接入后，可以通过在代码中添加注释的就就可以自动生成swagger-ui ，并在本地的web页面中可以直接复制出OpenAPI Schema
				- ![图片](https://mmbiz.qpic.cn/mmbiz_png/33P2FdAnjuicFRK8Q7SE9DN5cYfB1P31qE3VOwMd1AdwGMW4y36XXnCicmk93FGgTtrgjtDMiaQ1TPpQ68m8O6AhA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
			- #### 3. 通过网关获取接口定义
				- 第三种方式是通过服务端注册网关，天猫品牌行业这边中后台页面接口有统一的网关服务用于管理：服务注册，稳定性流量的监控，生成http服务等。使用网关的研发流程中，服务端开发可以通过提供的Idea插件或者是手动配置的在网关上完成注册。我们在内部和网关团队合作，打通了服务的网关信息并支持完成OpenAPI Schema的转换。
				- ### **schema到代码生成注入**
					- 在拿到接口的详细OpenAPI schema定义后，就可以自动化的生成数据请求所需要的前端相关代码，主要是以下几部分：
						- Model：每个接口的Request，Response，DTO都有对应的数据模型的TS定义，类型定义清晰明确，确保代码的健壮性。
						- Service：封装完整的数据请求服务，包括请求路径，类型，参数，异常处理等。（部分的高级组件内部会集成状态管理，只需要把数据请求服务作为参数传入）。
						- Hooks：在Service的基础上进一步集成React状态管理，直接用于可以对接Fusion等UI组件
						- Mock：本地的数据仿真服务，通过faker.js模拟数据，根据环境识别自动切换本地仿真数据请求
					- 除此以外，也可以根据业务的需要注入相关的业务埋点服务,稳定性监控等。具体的模版到代码的生成实现方案可以参考开源社区的优秀工具集，如：Kubb(https://kubb.dev/) 是专为现代 TypeScript 前端工程设计的 OpenAPI 代码生成器，通过解析 OpenAPI 3.x 规范自动生成完整类型安全的 API 客户端代码。
					- ![图片](https://mmbiz.qpic.cn/mmbiz_png/33P2FdAnjuicFRK8Q7SE9DN5cYfB1P31qa1RAqKBiaVKCaIB8ZGL52ibAnLatY2ymOZFq7ZhyeIricEib908phmZiahw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
					-
	- ## 代码拟合和调整
		- ### **业务代码拟合**
			- 以上我们已经获取到了AI生成的UI交互代码和交互数据相关的接口服务，实体模型和react hooks，下一步就是让AI根据仓库下这些的代码素材进行业务逻辑的拟合和拼装。
			- 拟合提示词
			  logseq.order-list-type:: number
			  collapsed:: true
				- logseq.order-list-type:: number
				  ```apl
				  // 此处省略通用的一些编码要求
				  ......
				  你是一个高级前端开发工程师，具有React组件和Hooks的深度开发经验
				  熟练编写和集成自定义 hooks 与组件
				  将用户提供的 React 组件代码与自定义 hooks 整合在一起。
				  偏爱使用${component_lib}组件，如果必要或用户要求，可以使用其他第三方库。
				  
				  ....
				  
				  ## 注意事项
				  确保整合代码的逻辑流畅和一致性
				  理解用户的主要目标，包括如何整合组件和 Hooks，确保整合后的代码符合最佳实践。
				  确保 Hooks 的方法名、引用路径以及类型定义的准确性和一致性。
				  导入 hook, model 的目录路径应该是相对于当前文件的路径，并加上 `generate`。
				  最终组件中使用的数据字段应该是根据请求返回的数据定义来的。
				  
				  // 此处省略具体的内容规划，结构格式要求和执行路径
				  ......
				  ```