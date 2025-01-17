# HOWTO

Here are recipes for running analyses on different platforms.
Before following these instructions, make sure you have completed installation and possible account setup detailed in [installation instructions](installation.md). Note that although running the pipeline directly with Cromwell is still possible, using [caper](https://github.com/ENCODE-DCC/caper) is the canonical, supported and official way to use ENCODE Uniform Processing Pipelines. The examples below use command `caper run`, which is the simplest way to run a single pipeline instance. For running multiple pipelines in production setting we recommend using caper server. To find details on setting up the server, refer to [caper documentation](https://github.com/ENCODE-DCC/caper/blob/master/DETAILS.md#usage).

Note that the files used in these examples are first restricted to reads from chromosome 19, and then further subsampled to 10000 reads. The cpu and memory resources reflect the size of inputs. For resource guidelines with full sized data, see discussion [here](reference.md#note-about-resources).

# CONTENTS

[Google Cloud](howto.md#google-cloud)  
[Truwl](howto.md#truwl)     
[Other Platforms](howto.md#other-platforms)  
[Splice Junctions](howto.md#splice-junctions)  


# RUNNING WORKFLOWS

## Google Cloud

Make sure you have completed the steps for installation and Google Cloud setup described in the [installation instructions](installation.md#google-cloud). The following assumes your Google Cloud project is `YOUR_PROJECT`, you have created a bucket `gs://YOUR_BUCKET_NAME`, and also directories `input`, `output` and `reference` in the bucket.
The goal is to run the pipeline with test data using Google Cloud Platform. Make sure caper version is `0.8.2.1` or newer. This is needed for docker image autodetection.

1. Launch a VM into your Google Cloud project, and connect to the instance.

2. Get the code and move into the code directory:

```bash
  git clone https://github.com/ENCODE-DCC/long-read-rna-seq-pipeline.git
  cd long-read-rna-seq-pipeline
```

3. Copy input and reference files into your bucket:

```bash
  gsutil cp gsutil cp test_data/chr19_test_10000_reads.fastq.gz gs://YOUR_BUCKET_NAME/input/
  gsutil cp test_data/GRCh38_no_alt_analysis_set_GCA_000001405.15_chr19_only.fasta.gz test_data/00-common_chr19_only.vcf.gz test_data/gencode.v24.annotation_chr19.gtf.gz gs://YOUR_BUCKET_NAME/reference/
```

4. Prepare the `input.json`. In the following template fill in the actual URI of your Google Cloud Bucket and save the file as `input.json` in the `long-read-rna-pipeline` directory.

```
{
    "long_read_rna_pipeline.fastqs" : ["gs://YOUR_BUCKET_NAME/input/chr19_test_10000_reads.fastq.gz"],
    "long_read_rna_pipeline.reference_genome" : "gs://YOUR_BUCKET_NAME/reference/GRCh38_no_alt_analysis_set_GCA_000001405.15_chr19_only.fasta.gz",
    "long_read_rna_pipeline.annotation" : "gs://YOUR_BUCKET_NAME/reference/gencode.v24.annotation_chr19.gtf.gz",
    "long_read_rna_pipeline.variants" : "gs://YOUR_BUCKET_NAME/reference/00-common_chr19_only.vcf.gz",
    "long_read_rna_pipeline.experiment_prefix" : "TEST_WORKFLOW",
    "long_read_rna_pipeline.input_type" : "pacbio",
    "long_read_rna_pipeline.genome_build" : "GRCh38_chr19",
    "long_read_rna_pipeline.annotation_name" : "gencode_V24_chr19",
    "long_read_rna_pipeline.small_task_resources" : {
        "cpu": 1,
        "memory_gb": 1,
        "disks": "local-disk 10 SSD"
    },
    "long_read_rna_pipeline.medium_task_resources" : {
        "cpu": 1,
        "memory_gb": 1,
        "disks": "local-disk 10 SSD"
    },
    "long_read_rna_pipeline.large_task_resources" : {
        "cpu": 1,
        "memory_gb": 1,
        "disks": "local-disk 10 SSD"
    },
    "long_read_rna_pipeline.xlarge_task_resources" : {
        "cpu": 1,
        "memory_gb": 1,
        "disks": "local-disk 10 SSD"
    }
}
```

5. Run the pipeline using caper:

```bash
  $ caper run long-read-rna-pipeline.wdl -i input.json -b gcp -m testrun_metadata.json
```


6. Run croo, to to make finding outputs easier:

```bash
  $ croo testrun_metadata.json --out-dir gs://[YOUR_BUCKET_NAME]/croo_out
```

This command will output into the bucket an HTML table, that shows the locations of the outputs nicely organized. Note that if your output bucket is not public, you need to be logged into your google account to be able to follow the links.

## Truwl

You can run this pipeline on [truwl.com](https://truwl.com/workflows/library/ENCODE%20Long%20read%20RNA-seq%20pipeline/v2.0.0). This provides a web interface that allows you to define inputs and parameters, run the job on GCP, and monitor progress in a ready-to-go environment. To run it you will need to create an account on the platform then request early access by emailing [info@truwl.com](mailto:info@truwl.com) to get the right permissions. You can see the 2-rep example case from this repo [here](https://truwl.com/workflows/library/ENCODE%20Long%20read%20RNA-seq%20pipeline/v2.0.0/instances/WF_308a61.8f.181f). The example job (or other jobs) can be forked to pre-populate the inputs for your own job.

If you do not run the pipeline on Truwl, you can still share your use-case/job on the platform by getting in touch at [info@truwl.com](mailto:info@truwl.com) and providing your inputs.json file.

## Other platforms

Running on other platforms is similar, because the caper takes care of the details for you. See [caper documentation](https://github.com/ENCODE-DCC/caper#installation) for further details.

## Using Singularity

Caper comes with built-in support for singularity with `--singularity` option. See [caper documentation](https://github.com/ENCODE-DCC/caper/blob/master/DETAILS.md) for more information.
