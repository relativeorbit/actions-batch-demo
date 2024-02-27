# actions-batch-demo
Demo repository for batch processing with Github Actions

Install the [GitHub actions CLI](https://cli.github.com) in order to easily run workflows from the command line. Alternatively you can manually run workflows from the 'Actions' repository tab. NOTE: you must be a member of this GitHub organization to run these workflows, or you can fork this repository and add your own secrets (see below).

## Run single workflow

```bash
gh -R relativeorbit/actions-batch-demo workflow run single-job.yml \
  -f reference=S1_136231_IW2_20200604T022312_VV_7C85-BURST \
  -f secondary=S1_136231_IW2_20200616T022313_VV_5D11-BURST
```

Note: the first time this worflow is run the conda environment is created (~3min) and then cached. Subsequent runs are faster because the environment does not need to be re-created.

It can be convenient to use custom names if you're running many workflows

```bash
gh -R relativeorbit/actions-batch-demo workflow run single-job.yml \
  -f reference=S1_136231_IW2_20200604T022312_VV_7C85-BURST \
  -f secondary=S1_136231_IW2_20200616T022313_VV_5D11-BURST \ 
  -f apply_water_mask=true \
  -f workflow_name=S1_136231_IW2_20200604_20200616
```

### Monitor job

You can go to the GitHub Repository's Actions tab (https://github.com/relativeorbit/actions-batch-demo/actions) or monitor from the command line:
```bash
gh -R relativeorbit/actions-batch-demo run list --workflow=single-job.yml
```

You can easily view logs referring to unique job ids reported by the command above

```bash
gh run view 8070732913 --log 
```

### Collect workflow outputs


