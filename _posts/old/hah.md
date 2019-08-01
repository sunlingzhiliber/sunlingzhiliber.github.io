
**Preregistration and reproducibility**

The assumption that researchers in Experimental Economics preferentially report statistically significant results is potentially more problematic.

data FAIR

**A framework for characterising and evaluating the effectiveness of environmental modelling**

Theoretical basis and  conceptual model 
Credibility of inputs
Testability of implementation
Match to observed behaviour
Treatment of uncertainty
End-user input
Input and output accessibility, and model transparency and traceability





**Integrating scientific cyberinfrastructures to improve reproducibility in computational hydrology: Example for HydroShare and GeoTrust**
作者：Bakinam T. Essawy
可计算环境模型的可再现是一个巨大挑战，他需要开放、可重用的代码、数据，详细的描述的工作流，稳定的环境，以此让别人，让peers能够验证已经发表的结果。
需要原始数据的文档说明，数据预处理脚本，模型输入，模型输出，所有的依赖环境。
提供了模型、数据的共享。
大量的应用，以modelflow的方式来复现可执行的工作流。
GeoTrust Sciunit-CLi提供了可重复计算的技术工作流程，
HydroShare提供了公共存储和元数据支持
pre-processing script 


**An overview of the model integration process: From pre-integration assessment to testing**

模型继承的步骤：
模型预继承评估 准备模型 模型的编排 数据互操作 测试

Data exchange and data manipulation are fundamental to integrated modeling systems (Argent, 2004; Leimbach and Jaeger,2005),


Ramamoorthy et al. (1992) point out that the main cost of integration is the effort required to detect and resolve inconsistencies in the coupled system, a cost that can be reduced if inconsistencies are detected and resolved prior to integration – that is, during the process of transforming the science design to a software design. 

数据互操作：
 数据的mediation 是由单独的组件模块实现，还是由参与者模型本身来调解数据。
 如果一个模型可以链接5个模型，则它应该包含每个组件的数据集实现。
 为每个模型实现数据集转换函数，当数量增加而缺乏良好的管理性后，这种方法不可扩展。


 Providing models via the web is not without challenges. As Brooking and Hunter (2013) point out, these challenges include managing large volumes of data, complexity of the required software platform, and computational demand for model execution are barriers that limit sharing and re-use of models over the web. Similarly, Nativi et al. (2013) point out developing a flexible architecture to link distributed components, minimizing interoperability agreements, optimizing performance, and securing availability in the long term are challenges in presenting models on the web. Presenting models using web pages also creates a challenge since model usage is constrained by how the web page presents it. For example, users may want to display model output as part of another application, which leads to providing a “model as a service”, making it available as a web service (Geller and Melton, 2008; Geller and Turner, 2007; Roman et al., 2009). This approach makes models and their output more accessible, more comparable, more scalable, and can be implemented using a variety of approaches (Nativi et al., 2013). One example of such a platform is eHabitat (Dubois et al., 2013) which is part of the European Union Digital Observatory for Protected Areas (DOPA20) project. In eHabitat, models and data sources are presented as a pool of OGC WPS services; using a web-based interface, users can select and assemble them like Lego blocks. Another example is the iPlant cyberinfrastructure (Goff et al., 2011) which provides several biology-related applications as a web service API so bioinformatics experts and software developers can embed them in their tools. Some modeling frameworks use purposely-designed semantic technology features to improve discoverability of models and related resources. For example, Galaxy (Goecks et al., 2010) e a computational framework for life sciences e uses tagging (labeling) to describe resources. During simulation, the system automatically tracks input datasets, tools used, parameter values, output datasets, and a series of analysis steps that it stores as history. The user can add annotations about analysis steps. The authors report that tagging supports reproducibility of experiments. Similarly, the system biology data and model platform - SEEK (Wolstencroft et al., 2015) e provides a web-based environment for day-to-day collaboration and public access. It uses the Resource Description Framework (RDF) to store metadata of resources that facilitate data and model exploration. All of these efforts indicate that incorporation of purposely-designed semantic technology features can improve discoverability of models beyond simple search operations.

 可发现性、可访问性和易用性是用户的基本需求。对于现有系统，可发现性通常仅限于组、文献和谷歌搜索中的本地知识。如上所述，文献中常常没有提供足够的细节来确定软件的重要特性，而这些特性是做出重用决策所必需的。值得肯定的是，我们看到了web作为部署建模组件的平台的趋势，这似乎是由于人们希望通过将模型作为服务提供来解决互操作性问题，从而消除不兼容的计算机语言和操作系统的问题。另一个好处是，基于web的部署还增强了模型的可发现性。


**Sharing and performance optimization of reproducible workflows in the cloud**

科学工作流在现代科学中扮演着重要的角色，因为它使科学家能够指定、共享和重用计算实验。为了使效益最大化，工作流需要支持它们所捕获的实验方法的重现性。由于科学家可以重新执行他人开发的实验，并迅速得出新的或改进的结果，可重复性使有效共享成为可能。然而，在实践中实现重现性是有问题的——以前的分析强调了由于输入数据、配置参数、工作流描述和用于实现工作流任务的软件的不受控制的更改而导致的问题。由此产生的问题被称为工作流衰减。

Recent research on workflows and workflow management systems has found that a large number of workflows cannot be reused, nor can they produce the same results over time [5]. This has been termed workflow decay [6] and stems from a variety of factors, including: the lack of an adequate workflow description, missing resources required to execute workflows such as data and services, and changes in the workflow execution environment [7].

我们的科学工作流程可重复性框架的完整描述，包括:工作流程建模、动态部署、图像管理和版本控制，

新的性能优化技术，增强我们的重现性框架，包括:
一种命名、创建和选择兼容任务图像的新算法，提高了准备运行工作流组件的可重用性，
支持工作流共享和优化工作流部署过程的可部署组件的多级缓存，
任务工件和依赖包的缓存，以支持创建映像的过程，
使用在本地和云环境上运行的真实和合成的科学工作流来验证和评估所提议的机制的一组实验。这些显示了工作流供应和制定的性能改进，以及工作流共享方面的改进

镜像~~~~





**Independent discussion sections for improving inferential reproducibility in published research**

虽然复制方法和结果是必要的，以证明重复性，这两个主要支柱的重复性是不够的。第三个支柱是在理论框架内的一致性解释，被称为`推理可重复性`2(图1)。

Enhancing inferential reproducibility is necessary to the appropriate determination of whether new knowledge merits advance along the translational continuum
