---
title: "FSATOOL: A useful tool to do the conformational sampling and trajectory analysis work for biomolecules"
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
publication: "*Journal of Computatioanl Chemistry, 41*(2)"
publication_short: "J. Comput. Chem"

abstract: "Reliable conformational sampling and trajectory analysis are always important to the study of the folding or binding mechanisms of biomolecules. Generally, one has to prepare many complicated parameters and follow a lot of steps to obtain the final data. The whole process is too complicated to new users. In this paper, we provide a convenient and user-friendly tool that is compatible to AMBER, called FSATOOL (fast sampling and analysis tool). FSATOOL has some useful features. First and the most important, the whole work is extremely simplified into two steps, one is the fast sampling procedure and the other is the trajectory analysis procedure. Second, it contains several powerful sampling methods for the simulation on GPU, including our previous mixing REMD method. The method combines the advantages of the biased and unbiased simulations. Finally, it extracts the dominant transition pathways automatically from the folding network by Markov state model. Users do not need to do the tedious intermediate steps by hand. To illustrate the usage of FSATOOL in practice, we perform one simulation for a RNA hairpin in explicit solvent. All the results are presented."

# Summary. An optional shortened abstract.
summary: Implementing the Mixing-REMD method by our developing FSATOOL program. It contains the sampling module and Markov State Model(MSM) which it's easy to use.

featured: false

# links:
# - name: ""
#   url: ""
#url_pdf: http://arxiv.org/pdf/1512.04133v1
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
  caption: 'Image credit: [**JCC**](https://unsplash.com/photos/jdD8gXaTZsc)'
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
---