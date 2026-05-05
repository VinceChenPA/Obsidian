Rules是「准则」，Memories是「记忆」，MCP是「触手」，Skill是「技能」，那Workflow就是「习惯」，几个动作串起来形成的一套固定流程。

**用一个类比串起来**

想象一个新来的全栈工程师。

Rules，是你给他的员工手册。公司用什么技术栈，代码怎么命名，PR怎么提，这些写在里面，他每天上班前翻一遍。

Memories，是他自己的工作笔记。「张三说过这个模块不要动」「上次部署出过一个奇怪的bug」。这些是工作中慢慢积累的经验，别人看不到，只有他自己知道。

MCP，是他工位上的那些工具。能登GitHub，能连数据库，能查Slack消息，能看Figma设计稿。没有这些工具他只能干瞪眼，有了这些他才能真正干活。

Skill，是他掌握的专业技能。怎么做code review，怎么写migration脚本，怎么排查内存泄漏。不是每天都用，但需要的时候必须拿得出来。

Workflow，是他的日常routine。早上先pull最新代码，跑一遍测试，开始写feature，写完跑lint，提PR，等review。这套流程固定下来，就是工作习惯。

五个概念不是互斥的，它们是协同工作的不同层面。一个配置良好的AI编程环境，应该是Rules打底，Memories辅助，MCP提供外部连接，Skill补充专业流程，Workflow串起日常动作。

**实际操作建议**

大多数人只需要先做好前两层就够了。

先写一份好的Rules文件。不用长，把技术栈、命名规范、目录结构、常用命令写清楚就行。这一步能解决80%的问题。然后让Memories自然积累，不用刻意管。等发现某个复杂流程需要反复跟AI解释的时候，把它抽成一个Skill。等需要AI去读外部数据或操作外部工具时，接一个MCP服务器。Workflow最后再搞，等前面几层都稳了再说。

不要一上来就把五层全配满。那样只会把context window塞爆，AI反而变笨。

**跨工具迁移**

这几个概念在不同工具里叫法不同，但底层逻辑是通的。Cursor的.cursorrules内容可以几乎原封不动搬到Windsurf的.windsurfrules里。Claude Code的CLAUDE.md也可以改个格式搬到Cursor里用。MCP本身就是跨平台的标准协议，同一个MCP服务器可以同时被Cursor、Windsurf、Claude Code调用。

唯一不太好迁移的是Memories，因为它是跟特定工具绑定的本地状态。但如果关键知识都已经写进了Rules和Skill，Memories丢了也不心疼。

各工具的概念对应关系整理如下。

Cursor，Rules对应.cursorrules和.mdc文件。Memories是最近加入的实验功能。MCP支持。没有独立的Skill概念，用高级Rules或社区方案替代。没有Workflow。

Windsurf，Rules对应.windsurf/rules/目录和.windsurfrules文件。Memories是Cascade自动生成的。MCP支持。Skill的概念融合在Rules的不同触发模式里。Workflow是声明式自动化流程。

Claude Code，Rules对应CLAUDE.md。Memories分为手动编写的CLAUDE.md和Auto Memory自动积累的笔记。MCP支持。Skill是独立的.claude/skills/目录。Workflow的功能分散在Hooks和Subagent里。

很多人配了一堆东西效果反而变差，就是因为把应该放在Skill里的东西塞进了Rules，或者该用MCP解决的问题用Memories来硬扛。

工具在变，概念在变，底层的设计逻辑没变。搞明白每一层在干嘛，就不会被花里胡哨的功能名搞晕了。