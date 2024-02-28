# actions-batch-demo
Demo repository for batch processing with Github Actions

NOTE: you must be a member of this GitHub organization to successfully run the commands below, or you can fork this repository and add your own secrets.

Install the [GitHub actions CLI](https://cli.github.com) in order to easily run workflows from the command line. Alternatively you can manually run workflows from the 'Actions' repository tab. 

This example workflow processes 2 public satellite images with open source software. The software uses multiprocessing to take advange of multiple CPU-cores, takes about 5 minutes to run, and generates about 30MB of output images.

## Run single workflow

```bash
gh -R relativeorbit/actions-batch-demo workflow run single-job.yml \
  -f reference=S1_136231_IW2_20200604T022312_VV_7C85-BURST \
  -f secondary=S1_136231_IW2_20200616T022313_VV_5D11-BURST
```

NOTE: the first time this worflow is run the conda environment is created (~3min) and then cached. Subsequent runs are faster because the environment does not need to be re-created.

NOTE: `-R relativeorbit/actions-batch-demo` is only required if you are not running the command from a local repository folder

It can be convenient to use custom names if you're running many workflows

```bash
cd actions-batch-demo

gh workflow run single-job.yml \
  -f reference=S1_136231_IW2_20200604T022312_VV_7C85-BURST \
  -f secondary=S1_136231_IW2_20200616T022313_VV_5D11-BURST \
  -f apply_water_mask=true \
  -f workflow_name=S1_136231_IW2_20200604_20200616
```

### Monitor job

You can go to the GitHub Repository's Actions tab (https://github.com/relativeorbit/actions-batch-demo/actions) or monitor from the command line:
```bash
gh run list --workflow=single-job.yml
```

You can easily view logs referring to unique workflow IDs reported by the command above

```bash
gh run view 8070732913 --log 
```

### Download workflow outputs

Our example workflow generates many files in a custom-named output folder `20200604_20200616` which is zipped and stored as a GitHub Actions "Artifact". We can retrieve and unzip this output for a given workflow ID: 

```bash
cd /tmp
gh -R relativeorbit/actions-batch-demo run download 8070991212
```

NOTE: artifact downloads use temporary signed urls that require GitHub authentication (https://docs.github.com/en/rest/actions/artifacts?apiVersion=2022-11-28#download-an-artifact). The above command performs authentication, follows, redirects, downloads and unzips for you.


## Batch computing with a re-useable workflow

Above we described a single workflow, which is similar to a 'serverless function'. It can be called with different inputs, is run on GitHub/Microsoft Azure VMs, and outputs temporarily stored in Blob storage. Next we take advantage of GitHub's 'resuable' actions to map many jobs (must be less than 256) to process a list of inputs in parallel.

The example used 2 satellite images, in fact there is a stack of these images and we can process pair-wise combinations. This job performs a search for compatible images (using a public API and image stack identifier), then maps image pairs as jobs executed by our existing workflow.

```bash
gh workflow run batch-job.yml -f burstId=064_136231_IW2
```

NOTE: An alternative approach would be to write your own script to loop over a range of inputs and fire off jobs by calling the original workflow (psuedocode below). However, it's can be advantages to keep many related jobs under the same 'workflow' and to be able to launch large processing queues from the GitHub.com interface!

```python
import os
pairsList = [(image1,image2), (image3,image4)]
for ref, sec in pairsList:
  os.system(f'''gh -R relativeorbit/actions-batch-demo workflow run single-job.yml \
  -f reference={ref} \
  -f secondary={sec}''')
```

### Monitor batch processing

```bash
gh run list --workflow=batch-job.yml
```

```
STATUS  TITLE           WORKFLOW  BRANCH  EVENT              ID          ELAPSED  AGE                 
*       064_136231_IW2  Batch     main    workflow_dispatch  8072431427  8m1s     about 8 minutes ago
```

```bash
gh run view 8072431427
# Or specific job of a workflow
gh run view --job=22054171572
```

### Download artifacts from Batch workflow 

```bash
cd tmp
gh -R relativeorbit/actions-batch-demo run download 8072431427 
```

NOTE: you can do this even if not all jobs have completed! It can take a while to download all artifacts for workflow runs, you can use filters to download only certain artifacts.


### Scaling up

So far we've run a workflow that processes 15 image pairs in parallel. Since each job uses 4vCPU and 16GB, we effectively have used a compute cluster size of 60vCPU, which is more than a typical laptop. Also we didn't have our laptop fan going full blast and this completed in less than 9 minutes! Still, larger workflows are common place. Next we try a set of 157 images:

```bash
gh workflow run batch-job.yml -f burstId=115_245676_IW2
```

Note: most accounts have access to 20 concurrent runners. Here we have 156 jobs, so this workflow will take a bit longer as jobs get queued (156/20)*9min ~ 70min. If you have an academic or 'pro' account you have 


```
gh run download 8073568114 --dir /tmp --name "20200420_20200502"
gh run download 8073568114 --dir /tmp --pattern "*20200220*"
```

## Configuration

* The workflow requires the following Actions secrets. Links below to set up free accounts with these data providers.
  * `EARTHDATA_USERNAME` & `EARTHDATA_PASSWORD` (to download S1 Images from ASF DAAC https://urs.earthdata.nasa.gov)
  * `ESA_USERNAME` & `ESA_PASSWORD` (to download Sentinel-1 precise orbits from https://dataspace.copernicus.eu)


## Ackowledgments
[University of Washington eScience Winter Incubator 2024](https://escience.washington.edu/incubator-24-glacial-lakes/)