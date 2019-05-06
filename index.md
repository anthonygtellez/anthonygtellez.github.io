---
layout: article
mode: immersive
title:  Machine Learning, Cyber Security  & Cloud Ops.
subtitle:  Testing
header:
  theme: dark
article_header:
  type: overlay
  theme: dark
  background_color: '#203028'
  background_image:
    gradient: 'linear-gradient(135deg, rgba(34, 139, 87 , .4), rgba(139, 34, 139, .4))'
    src: http://tellez.sfo2.digitaloceanspaces.com/earth-nasa-hero.jpg
---

### Anthony G. Tellez

A personal website to aggregate public presentations, blog posts in the domains of machine learning, cyber security and cloud ops. 

[Conference Presentations](/archive.html?tag=Conference){:.button.button--primary.button--rounded.button--lg}


### Interests


```chart
{
  "type": "radar",
  "data": {
    "labels": [
      "Machine Learning",
      "Cryptography",
      "Visualization",
      "Deep Learning",
      "Japanese",
      "Traveling",
      "Cooking"
    ],
    "datasets": [
      {
        "label": "My Interests",
        "backgroundColor": "rgba(255,99,132,0.2)",
        "borderColor": "rgba(255,99,132,1)",
        "pointBackgroundColor": "rgba(255,99,132,1)",
        "pointBorderColor": "#fff",
        "pointHoverBackgroundColor": "#fff",
        "pointHoverBorderColor": "rgba(255,99,132,1)",
        "data": [
          90,
          80,
          100,
          80,
          96,
          98,
          80
        ]
      }
    ]
  },
  "options": {}
}
```

```mermaid
graph TD
A[Data] --> |Split Data| B(Testing Dataset)
A[Data] --> |Split Data| C(Training Dataset)
B --> D(Algorithm)
C --> E(Evaluation)
D --> E(Evaluation)
E --> F(Model) 
F --> G(Prediction)
H(Production Data) --> F(Model)

```