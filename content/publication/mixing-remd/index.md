---
title: "Combining the Biased and Unbiased Sampling Strategy into One Convenient Free Energy Calculation Method"
authors:
- admin
- Changjun Chen*
date: "2019-09-01T00:00:00Z"
doi: ""

# Schedule page publish date (NOT publication's date).
publishDate: "2019-01-01T00:00:00Z"

# Publication type.
# Legend: 0 = Uncategorized; 1 = Conference paper; 2 = Journal article;
# 3 = Preprint / Working Paper; 4 = Report; 5 = Book; 6 = Book section;
# 7 = Thesis; 8 = Patent
publication_types: ["2"]

# Publication name and optional abbreviated publication name.
publication: "*Journal of Computatioanl Chemistry, 40*(1806-1815)"
publication_short: "J. Comput. Chem"

abstract: "Constructing a free energy landscape for a large molecule is difficult. One has to use either a high temperature or a strong driving force to enhance the sampling on the free energy barriers. In this work, we propose a mixed method that combines these two kinds of acceleration strategies into one simulation. First, it applies an adaptive biasing potential to some replicas of the molecule. These replicas are particularly accelerated in a collective variable space. Second, it places some unbiased and exchangeable replicas at various temperature levels. These replicas generate unbiased sampling data in the canonical ensemble. To improve the sampling efficiency, biased replicas transfer their state variables to the unbiased replicas after equilibrium by Monte Carlo trial moves. In comparison to previous integrated methods, it is more convenient for users. It does not need an initial reference biasing potential to guide the sampling of the molecule. And it is also unnecessary to insert many replicas for the requirement of passing the free energy barriers. The free energy calculation is accomplished in a single stage. It samples the data as fast as a biased simulation and it processes the data as simple as an unbiased simulation. The method provides a minimalist approach to the construction of the free energy landscape."

# Summary. An optional shortened abstract.
summary: A mixing-REMD method to fast sample the configurations of protein and obtain the free energy landscape. It combines the REMD and Metadynamis/ABMD which makes the sampling method more powerful.

featured: false

# links:
# - name: ""
#   url: ""
# url_pdf: http://arxiv.org/pdf/1512.04133v1
url_code: ''
url_dataset: ''
url_poster: ''
url_project: ''
url_slides: ''
url_source: ''
url_video: ''

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder. 
image:
  caption: 'Image credit: [**JCC**](test)'
  focal_point: ""
  preview_only: false

# Associated Projects (optional).
#   Associate this publication with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `internal-project` references `content/project/internal-project/index.md`.
#   Otherwise, set `projects: []`.
projects: []

# Slides (optional).
#   Associate this publication with Markdown slides.
#   Simply enter your slide deck's filename without extension.
#   E.g. `slides: "example"` references `content/slides/example/index.md`.
#   Otherwise, set `slides: ""`.
slides: example
---
