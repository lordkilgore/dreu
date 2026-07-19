**Student:** Phoenix Wilson

**Mentor:** Kristina Gligoric

# Week 2

**Dates:** 07-12 to 07-18

## Goals
- Set up workflow
- Build toy dataset containing examples with clearly hierarchical variables
- Load Llama 3.1 8B
- Integrate Forward Hooks
- Implement Lens


## Approach and Implementation
Setting up my workflow, I wrestled with accessing the cluster (which was a persistent, widespread issue throughout the week; sysadmin team had to be alerted) and then set up my conda environment. However, my conda environment was corrupted at some point, so I made the move to migrate to the more modern project manager, `uv`. 

After setting up the project repository and environment, I developed the toy dataset generation code. The design philosophy was to be as modular as possible, supporting config files with arbitrary size variable sets. I decided to generate data using the shape-table template specified in last week's log; the associated config file is shown below:

```shape_incline_toy.yml

template: "A {} is placed on a table which has a {} degree decline. Will the object slide off?"
variables:
  shapes: ["tetrahedron", "cube", "octahedron", "dodecahedron", "icosahedron"]
  decline: ["0", "15", "30", "45", "60", "75", "90"]
out_dir: "../data"
context_name: "shape_incline"
```





## Results



## Notes


