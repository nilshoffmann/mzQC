## Multi-Run (i.e. sets) Example of mzQC
Here, we describe an mzQC JSON document used to convey QC data which is computed on a set of runs, i.e.
is **only interpretable in the context of this set** (group).
Of course, QC metrics which refer to each run individually can also be stored, also in the same mzQC file
(see our example `individual-runs.mzQC.md` on how to do that), but this example is about group/set metrics.

Find the complete example file at the bottom of this document or in the example folder.

The basic structure of our mzQC file is identical to the `individual-runs.mzQC` example, i.e.
the documents main anchor is between the outer curly brackets:
```
{ "mzQC":
  {
    ...
  }
}
```

Within this main anchor, there are usually the following sections:
a) general information about the file,
```
    "version": "1.0.0",
    "creationDate": "2020-12-21T11:56:34",
    "contactName": "Chris Bielow",
    "contactAddress": "chris.bielow@bsc.fu-berlin.de",
    "description": "A simple mzQC file containing information for sets of runs.",
```

b) reference information for controlled vocabularies (cv) at the bottom, 
```
    "controlledVocabularies": [
      {
        "name": "Proteomics Standards Initiative Mass Spectrometry Ontology",
        "uri": "https://github.com/HUPO-PSI/psi-ms-CV/releases/download/v4.1.84/psi-ms.obo",
        "version": "4.1.84"
      }
    ] 
```
and (now in addition or as replacement) to the `runQualities` of the `individual-runs.mzQC` we have
c) information about the QC metrics computed on **a set of runs**.
```
    "setQualities": [
      {
        ...
      }
    ]
```
In fact, `setQualities` can contain one or more `setQuality` objects, each defining **a different** set of runs.
E.g. if you have three technical replicates for two conditions and a total of six runs, you might want to subsume three runs into a set, one set for each condition and report the groups' average m/z deviation of identifications, or the percentage of total intensity attributable to contaminants. 
Each `setQuality` object is an element of a JSON array, thus it is not explicitly named (i.e. there is no "setQuality" key in the mzQC file). 
However, for better organisation and better visualisation, you can add labels. These labes should be expressive and meaningful (e.g. "healthy"), 
and unique to the file, (e.g. if there are several healthy groups, they cannot all be labeled "healthy").
For the purpose of this example, we will use **three** sets, or more precise `setQuality` objects, from the two groups of our experimental design(`healthy`,`disease`) and one for all runs looked at together:

1. the **healthy** set: `tr1_healthy, tr2_healthy, tr3_healthy`
2. the **diseased** set: `tr1_diseased, tr2_diseased, tr3_diseased`
3. the **all** set: `tr1_healthy, tr2_healthy, tr3_healthy, tr1_diseased, tr2_diseased, tr3_diseased`

How you define (and label) each set, is up to you and depends on your experimental design and the kind of comparisons you want to make.

A `setQuality` represents QC data that must be viewed in the context of all the runs of this set/group. 
I.e. the data is only valid within the context of the runs it comprises. 
E.g. it would be invalid to define a set of three runs and report their individual MS1 scan counts as a 3-tuple -- because this information can clearly be attributed to individual runs and thus belongs in three separate `runQuality` objects, rather than a single `setQuality`.
Similar to `runQuality`, a `setQuality` also contains `metadata` about the set of runs (its input file**s**, the software used, etc). 
But since we have two groups, and want use metrics that compare both groups, we will also need a thrid set for _all_ contributing runs. 
You can give the set a unique name using the `label` attribute of the `setQuality` object. Here is how a `setQuality` object looks like:
```
      {
        "metadata": {
          "label": "healthy"
          "inputFiles": 
            ...
        },
        "qualityMetrics": [
            ...
        ]
      }
```
The `inputFiles` consist of an array of `inputFile` objects, describing the source files with structured information about the file's name, format, location and other properties, defined via cv terms. 
```
          "inputFiles": [
            {
              "name": "tr1_healthy",
              "location": "c:\msdata\techRep1_healthy.mzML",
              ...
            },
            {
              "name": "tr2_healthy",
              "location": "c:\msdata\techRep2_healthy.mzML",
              ...
            },
            {
              "name": "tr3_healthy",
              "location": "c:\msdata\techRep3_healthy.mzML",
              ...
            }
          ]
```
The `inputFile` object is only sketched here. It can contain a lot more information, such as file format and further properties. See the full example below or `individual-runs.mzQC` for details.

In `qualityMetrics`, we will store the actual QC information for a particular `setQuality`. 
Each `qualityMetric` has an `accession` and the corresponding `name` as defined by the QC controlled vocabulary (see `qc-cv.obo`). 
They should be represented exactly as stated in the .obo file. 
The `value` carries the actual information and can be either a single value, a tuple of values, a matrix or table. 
Below, we will look at single values and tables.

First, we compute the average m/z deviation of all identifications in each group, and report this in the a metric inside each group's `qualityMetrics` array. 
At this point, we have two sets defined with two labels. 
We compute this metric for each set, in our case for the `healthy` as well as the `diseased` set, but not for the `all` set (because we want to keep the example small). 
But in general, what metrics you compute is up to you.

Next, we will compare the groups, or rather investigate how well the recorded _protein group raw intensities_ are separating the two groups. 
This can be achieved with a principal component analysis (PCA), so we will report the resulting matrix as a metric and then visualise. 
The `setQuality` where this PCA metric will be stored, references **all** runs as input files, as we will consider the protein group raw intensities of all runs.
The input table for a such a PCA computation can be found in MaxQuant's proteinGroups.txt output file.
There, the protein group raw intensities are already aggregated on group level, thus we can't reference each run but must stick with the grouping, which is no probelem, since we have already declared and labeled the two groups as sets. 
Now your QC software can derive a new table using PCA, where each group is represented by PCA coordinates.

First, let's see what the PCA plot would look like:
![ Typically, the first two PCA dimensions are plotted, as shown here: Each data point in the plot represents one set(group), e.g. `diseased` or `healthy`.](figures/MultiSet_PCA.png)
Now, let's look at the mzQC data which allows you to create this plot: a 'PCA table' metric to store the computed _priciple components_ from the PCA (the first 5 principal components for each group represented by the 'PCA table' column terms `MS:4000081`-`MS:4000085`).
```
    "setQualities": [
      ...,
      {
        ...,
        
        "qualityMetrics": [
        {
            "accession": "MS:4000090",
            "name": "PCA of MaxQuant's protein group raw intensities",
            "value": {
                "MS:4000086": ["healthy", "diseased"],
                "MS:4000081": [47.22, -30.22],
                "MS:4000082": [29.1, -36.5],
                "MS:4000083": [3.8, -7.3],
                "MS:4000084": [-7.7, 5.55],
                "MS:4000085": [140.6, -64.1]
            }
        }
      ...,
    ]
```
As you've noticed, we use the `MS:4000086` column to refer to the respective groups. 
The labels used in that column must correspond to defined sets in the file, 
and as you probably guessed already, 
the combination of `inputFiles` in the all set must correspond to the union of `inputFiles` of the referenced sets. 
This representation then makes it easy to plot the PCA results in the visualisation framework of your choice.

### This is the mzQC file once again, in full:
**[set-of-runs.mzQC](examples/set-of-runs.mzQC)**
