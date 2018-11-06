From: Michal Januszewski <mjanusz@google.com>
To: Takemura, Shin-ya <takemuras@janelia.hhmi.org>
Cc: Plaza, Stephen <plazas@janelia.hhmi.org>; Berg, Stuart <bergs@janelia.hhmi.org>; Rivlin, Pat <rivlinp@janelia.hhmi.org>; Viren Jain <viren@google.com>
Date: Wednesday, October 31, 2018 at 11:24 AM

Hi everyone,

Thanks for the false split examples, this was very helpful! I've looked into these cases in some detail. The good news is that a new segmentation I already have solves 11/13 of the cases. The more tricky part is figuring out how to best integrate this into the current reconstruction. What follows is some background information on what I did, and some initial thoughts on how you might be able to use this data.

- The new segmentation (let's call it v2a) is based on a model trained on a number of sparsely traced bodies that I manually agglomerated from the current hemibrain base segmentation. I then used your 100+ sparsely traced bodies to select a best-performing checkpoint as measured by our skeleton metrics. The segmentation is at 32nm only and is available at 274750196357:hemibrain:seg_v2a_32nm_19315397 for your reference.

- I then looked at the voxel overlap between components in v2a and in your 38c6ad segmentation, and when a component in v2a overlapped more than 2 components in 38c6ad, subject to a minimum criterion for number of overlapping voxels, I created a tentative "merge" decision for these. This results in the following files uploaded to flyem-hemibrain-derived/seg_38c6ad:
  - overlap_groups_38c6ad_v3.txt.gz: sets of supervoxel IDs (from 38c6ad) to be merged; every line corresponds to one overlapping segment in v2a
  - overlap_groups_38c6ad_ccs_v3.gz: connected components of the merge graph defined by the previous file; one connected component per line
  - overlap_groups_38c6ad_ccs_dpoints_v3.csv.gz: decision points (id1, id2, xyz location in 32 nm coordinates) corresponding to the merge decisions above, in case these are helpful for your proofreading tools

The tricky part, as you will see, are some false merges that result from this process. These can be caused by failures of the new (v2a) segmentation (particularly caused by CB-CB mergers and due to membrane gaps), but also due to preexisting mergers in 38c6ad (some of which might be cleavable).

Please take a look at the files I shared and let me know if think you will be able to use them. If not, I can also try to do stronger filtering (e.g. require more overlapping voxels), but this will also remove some of the good merge decisions.

These suggested merge decisions have NOT been filtered to keep CBs separate or to take into account your focused proofreading decisions made so far. Doing so might be a way to reduce the false positive decisions. I imagine you might also want to filter them e.g. synaptic relevance (two or more supervoxels in the group with a synaptic contact)

In terms of the workflow, as I said before, it probably makes most sense to look at aggregated components (according to _ccs_v3 if workable, or _v3 otherwise). To deal with cases where the new components are clearly merged, one would need the ability to cleave within this workflow (this is particularly the case where the merger is caused by an underlying problem in 38c6ad and so the new aggregated component actually provides valuable information). If you need merge scores for this, computing IoU scores between the two segmentations locally for each decisions point could be one way to do it.

Please let me know what you think and if you need more data from us for this,
Michal
