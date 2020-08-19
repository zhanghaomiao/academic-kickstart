---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "PymolCheatsheet"
subtitle: ""
summary: "PYMOL Cheatsheet for visulize the structure of biomolecule"
authors: []
tags: [visualize]
categories: []
date: 2020-08-12T11:36:48+08:00
lastmod: 2020-08-12T11:36:48+08:00
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: ""
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---

## Introduction

- Select around:
  - select active, byres all with 5 of ligands
  - select ligand_water, ((ligands)around 3.2) and (resn HOH)

- Change radius
  - alter active_water, vdw=0.5
  - rebuild

- color and cartoon setting
  - set cartoon_color, slate
  - set transparency 0.4
  - set cartoon_fancy_helices, 1
  - set cartoon_highlight_color, grey50
  - bg_color, white
  - set antialias, 1

- load netcdf
  - load rna.prmtop
  - load_traj min2.rst, rna, 0, nc
  - load rna.prmtop, traj
  - load_traj prod_2us.nc, traj, 0, nc, start=8215, stop=8215

- Create from an object
  - create name, (selection), [, source_state [, target_state]]

- remove solvent and ions
  - remove solvent
  - remove inorganic

- Hydrogen bond
  - distance test, interface, nucleic, mode=2

- align with specified residue
  - align 1cfd_unbound///1-68/, 1cll_bound///1-68/

- Draw RNA dssr block
  - set ambient, 0.6  
  - dssr_block block_file=face+edge   block_color=C yellow

- Rename an object
  set_name rna rna01

- Print resName of an object
  - iterate bca. all , print (resn) [Iterate - PyMOLWiki](https://pymolwiki.org/index.php/Iterate)
  - Python API save to a name

    ```python
    myspace = {"resname": []}
    cmd.iterate("bca. all", "resname.append(resn)", space=myspace)
    ```

- Get XYZ of a atom
  - `cmd.get_coords("/1cll///20/CA")`

- Calcualte helix angle between
  - `angle_between_helices 1cll///71-76/, 1cll///86-92/, visualize=1`

- Calcualte helix every frame

  ```python
  for i in range(30):
    pymol.cmd.set("state", i+1)
    angle.append(anglebetweenhelices.angle_between_helices("1dmo///71-76/", "1dmo///86-92/"))
  ```
