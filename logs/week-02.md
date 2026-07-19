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

After setting up the project repository and environment, I developed the toy dataset generation code. The design philosophy was to be as modular as possible, supporting config files with arbitrary size variable sets. I decided to generate data using the shape-decline template specified in last week's log; the associated config file is shown below:

```shape_incline_toy.yml

template: "A {} is placed on a table which has a {} degree decline. Will the object slide off?"
variables:
  shapes: ["tetrahedron", "cube", "octahedron", "dodecahedron", "icosahedron"]
  decline: ["0", "15", "30", "45", "60", "75", "90"]
out_dir: "../data"
context_name: "shape_incline"
```

Once I had the data, I integrated Llama 3.1 8B support into my codebase. Actually loading the model was a technical challenge, as compute is limited on the cluster. In future experiments, GPU access may be necessary, however for the simple forward passes I did to debug, CPU with ~40G of memory sufficed. After the model was integrated, I developed a residual stream listener which registers forward hooks in the model to track outputs after each transformer layer.

Finally, I started implementing a logits lens, an approach to lensing which uses the outputs of intermediate transformer layers to output model logits. These intermediate representations were sourced from the listener, but parallelizing the lens across the entire dataset led to uninterpretable results which disagreed with how the model behaved during standard inferencing. For example, using the logits computed using the last layer representation from the listener did not match the logits output by the model's forward pass. This work will have to continue into the next week. 

## Results
- Project workflow set up using `uv` project manager.
- Toy dataset of 35 examples created of the form `A {} is placed on a table which has a {} degree decline. Will the object slide off?`
- Llama 3.1 8B support integrated into codebase with forward hooks to record residual stream
- Logit lens scaffolding implemented

This week was ripe with technical challenges as I was constantly being kicked off the cluster. However, going into next week, this should be less troublesome now that I know how to resolve the common causes behind these challenges.

## Notes


