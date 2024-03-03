---
# Leave the homepage title empty to use the site title
title:
date: 20240130
type: landing

sections:
  - block: hero
    content:
      title: |
        实验室概况
      image:
        filename: welcome.jpg
      text: |
        <br>
        
        实验室主要研究人工智能系统的安全性与人工智能技术在内容安全中的应用。主要研究方向包括（1）人工智能模型安全：深度神经网络版权溯源和完整性检验、模型水印和指纹的性能界限分析。（2）人工智能生成内容安全：具备可迁移性的深度伪造内容识别、生成多媒体内容模型的版权安全性分析。（3）生物特征信息的分析：基于大型预训练的唇读模型、不依赖预设内容的唇语特征身份识别等。实验室近年在相关领域国际顶刊顶会如IEEE TIFS，IEEE TCSVT，ICCV，IJCAI，AAAI等发表论文50余篇，多次承担国家自然科学基金研究，参与华为、奇安信、蚂蚁等横向课题数项。
        
  
  - block: collection
    content:
      title: 最新成果
      subtitle:
      text:
      count: 3
      filters:
        author: ''
        category: ''
        exclude_featured: false
        publication_type: ''
        tag: ''
      offset: 0
      order: desc
      page_type: publication
    design:
      view: card
      columns: '1'
  
  - block: markdown
    content:
      title:
      subtitle:
      text: |
        {{% cta cta_link="./member/" cta_text="实验室成员 →" %}}
    design:
      columns: '1'
---