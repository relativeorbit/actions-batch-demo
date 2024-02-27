# actions-batch-demo
Demo repository for batch processing with Github Actions

Install the [GitHub actions CLI](https://cli.github.com) in order to easily run workflows from the command line. Alternatively you can manually run workflows from the 'Actions' repository tab. NOTE: you must be a member of this GitHub organization to run these workflows, or you can fork this repository and add your own secrets (see below).

This example workflow processes 2 public satellite images with open source software. The software uses multiprocessing to take advange of multiple CPU-cores, takes about 5 minutes to run, and generates about 30MB of output images.

## Run single workflow

```bash
gh -R relativeorbit/actions-batch-demo workflow run single-job.yml \
  -f reference=S1_136231_IW2_20200604T022312_VV_7C85-BURST \
  -f secondary=S1_136231_IW2_20200616T022313_VV_5D11-BURST \
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

You can easily view logs referring to unique workflow IDs reported by the command above

```bash
gh -R relativeorbit/actions-batch-demo run view 8070732913 --log 
```

### Download workflow outputs

Our example workflow generates many files in a custom-named output folder `20200604_20200616` which is zipped and stored as a GitHub Actions "Artifact". We can retrieve and unzip this output for a given workflow ID: 

```bash
gh -R relativeorbit/actions-batch-demo run download 8070991212
```

Note: artifact downloads use temporary signed urls that require GitHub authentication (https://docs.github.com/en/rest/actions/artifacts?apiVersion=2022-11-28#download-an-artifact). The above command performs authentication, follows, redirects, downloads and unzips for you.

## Batch computing with a re-useable workflow

Above we described a single workflow, which is similar to a 'serverless function'. It can be called with different inputs, is run on GitHub/Microsoft Azure VMs, and outputs temporarily stored in Blob storage. Next we take advantage of GitHub's 'resuable' actions to map many jobs (must be less than 256) to process a list of inputs in parallel.

The example used 2 satellite images, in fact there is a stack of these images and we can process pair-wise combinations. This job performs a search for compatible images (using a public API and image stack identifier), then maps image pairs as jobs executed by our existing workflow.

```bash
gh -R relativeorbit/actions-batch-demo workflow run batch-job.yml \
  -f burstId=S1_136231_IW2
```