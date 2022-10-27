# wootrex bio API Documentation

## Oligo Design

The Oligo Design bioinformatics software package and API provide a simple and intuitive method to create single stranded oligonucleotide sequences that, when pooled together, assemble into the requested sequence. It also provides an ability to take a library of sequences with homologous and variable regions and create a design across the entire library that minimizes the number of homologous region oligos, while also preserving the required variable region oligos. Primers for amplification are additionally produced and recycled where necessary. Both the oligo design and the primer design are presented with their respective plate maps for ease of automation in the wet lab. The output is easily consumed by all major oligo manufacturers such as IDT, Twist, etc. 

### Parameters
The API accepts a JSON object containing the following:

#### API
A required string specifying which API is being requested. Valid strings are: "oligo_design", "sequence_complexity", "sequence_design"

#### Required Parameters
**required_parameters**: A required JSON object containing all three of the following entries:

###### Sequences
**sequences**: A list of JSON objects of sequence records that include the following:

1. **sequence_name**: String, a unique name for each sequence.
2. **sequence**: String, ambiguous/unambiguous DNA sequence.
3. **primer_length**: Integer, number of base pairs for the amplification primer.

###### Oligo Design Parameters
**oligo_design_parameters**: A JSON object containing the following:

1. **minimum_length**: Integer, default 30
The minimum size in base pairs allowed for any oligo in the design. 
2. **target_length**: Integer, default 50 
The target size in base pairs for an oligo to start with during design. 
3. **maximum_length**: Integer, default 60 
The maximum size in base pairs allowed for any oligo in the design. 
4. **minimum_overlap**: Integer, default 15
The minimum number of basepairs allowed for any given overlap between 3’ and 5’ paired oligos.
5. **maximum_overlap**: Integer, default 25
The maximum number of basepairs allowed for any given overlap between 3’ and 5’ paired oligos.

###### Primer Design Parameters 
**primer_design_parameters**: A JSON object containing the following:

1. **target_primer_length**: Integer, default 20
The minimum size in base pairs allowed for any oligo in the design. 
2. **tm_optimize_primers**: Boolean, default True
Wether or not to melting temperature (TM) optimize the amplification primers. 
3. **target_primer_tm_temp**: Integer, default 60
The temperature (in degrees Celsius) at wich PCA is performed. This is only used if **tm_optimize_primers** is **True**


#### Optional Parameters
**optional_parameters**: An optional JSON object containing any of the following:

1. **design_type**: String, default "maximize_recycling".
Specifies the type of Oligo Design to perform. Valid types are: "maximize_recycling", "minimize_complexity", "gapped_design", "standard"
2. **design_name**: String, default "design01"
User specified name for the design. The files emailed will have this name.
3. **recycle_oligos**: Boolean, default True.
Perform oligo recycling across the entire set of sequences provided in Sequence Records.
4. **source_plate_size**: Integer, default 96.
Number of wells of the oligo plate to order from the oligo manufacturer. Currently must be either 96 or 384. 
5. **destination_plate_size**: Integer, default 96.
Number of wells of the pooling plate. Currently must be either 96 or 384. 
6. **row_major**: Boolean, default True
Provide the list of oligos in row first (A1, A2, A3, etc) order if True, otherwise provide this list in column first order (A1, B1, C1, etc).
7. **emails**: String, default is empty
A comma separated list of emails to which to send the design files.
8. **partition_identity**: Float, default 0.95
Used only for Design Type = "maximize_recycling". This is used to help inform the Oligo Design about how best to partition the sequences for maximizing the recycling. Tuning this parameter will affect the overall recycle efficiency. This should be a number between 0.0 and 1.0.
9. **min_zscore_cutoff**: Float, default -4.0
Used only for Design Type = "minimize_complexity". This is used to help inform the Oligo Design how to identify outlier oligo pairings when estimating the ΔG for each oligo pair. Lower (more negative) values will result in less overall complex regions identified. 
10. **temp**: Float, default 60.0
Used only for Design Type = "minimize_complexity". This is used in the ΔG calculations and should correspond to the target PCA temperature (degrees Celsius). 
11. **optimize_overlap_tm**: Boolean, default False (NOT IMPLEMENTED AT API LEVEL YET)
Make oligo overlaps TM optimized to **overlap_tm_target_temp**
12. **overlap_tm_target_temp**: Float, default 60.0 (NOT IMPLEMENTED AT API LEVEL YET)
This should correspond to the target PCA temperature (degrees Celsius). 

### Output
An object that contains the oligo and primer designs, plate maps, and visualization  in JSON. See below for an example of the JSON output.

### Error Handling
If the design parameters are invalid or result in an invalid design (low success of assembly, or doesn’t match parameters) verbose errors are returned for resolution.

### Use Cases
#### Homology Path
- Oligo Design is fully implemented in the Ninth Bio Homology Path software suite
#### RESTful Web Service
- Secure network interface
- Could be on the internet or private network
- Could take JSON as input and output
- Could interact with customer databases
#### Command Line
- Same advantages as above, but additionally can run on a network isolated computer


### Technical Examples

Below is an example of a command line to POST a job request to the Oligo Design API.

```
curl -0 -X POST "https://homologypath.com/router/" \
-H "Authorization: Token <YOUR_API_TOKEN_HERE>" \
-H "Content-Type: application/json" \
-d \
'
{
    "api":"oligo_design",
    "required_parameters":{
       "sequences":[
          {
             "sequence_name":"example1",
             "sequence":"ATGGGGACCGGATCCGGATCGGGATCTCGCGCCCGGATCCGGATCGGAAAGCGAAA",
             "primer_length":"20"
          },
          {
             "sequence_name":"example2",
             "sequence":"ATGCCGGACGGATCCGGATCTCGCGGGATCTCGCACCGGATCCGGATCGTCCGGATCGCGAA",
             "primer_length":"20"
          }
       ],
       "oligo_design_parameters":{
          "minimum_length":"30",
          "target_length":"50",
          "maximum_length":"60",
          "minimum_overlap":"15",
          "maximum_overlap":"25"
       },
       "primer_design_parameters":{
          "target_primer_length":"20",
          "target_primer_tm_temp":"60",
          "tm_optimize_primers":"true"
       }
    },
    "optional_parameters":{
       "design_type":"maximize_recycling",
       "design_name":"design01",
       "source_plate_size":"384",
       "destination_plate_size":"384",
       "emails":"your@email.com",
       "row_major":"true",
       "recycle_oligos":"true",
       "partition_identity":"0.95"
    }
 }'

```

A successful response will be a JSON object containing the `job_id` of the requested Oligo Design similar to the following:

```
{"job_id":"b2816ba3-3390-437b-a37c-03fc1238db94"}
```

This `job_id` can be used to POST a polling request to the API for information about the job, for example:

```
curl -0 -X POST "https://homologypath.com/router/" \
-H "Authorization: Token <YOUR_API_TOKEN_HERE>" \
-H "Content-Type: application/json" \
-d \
'{"api": "oligo_design", "job_id":"b2816ba3-3390-437b-a37c-03fc1238db94"}'
```

Depending on how large the request was, this polling request may need to be done periodically until it's completed the calculation. Once the design is complete, there will be a `Finished` field and the response should look similar to the following:

```
{
  "Design": {
    "Oligo Order Form": [
      [
        "Plate Name",
        "Well Position",
        "Sequence Name",
        "Sequence",
        "Scale",
        "Purification",
        "Normalization Style",
        "Quantity (nmoles)",
        "Concentration (µM)",
        "Volume (µL)",
        "Buffer",
        "Notes"
      ],
      [
        "design01 oligo plate 1",
        "A01",
        "example1_20221026-215035_0",
        "ATGGGGACCGGATCCGGATCGGGATCTCGCGCCCGGATCCGGATCGGAAAGCGAAA",
        "25 nmole DNA Plate Oligo",
        "Standard Desalting",
        "Full Yield",
        "100",
        "100",
        "100",
        "",
        ""
      ],
      [
        "design01 oligo plate 1",
        "A02",
        "example2_20221026-215035_0",
        "ATGCCGGACGGATCCGGATCTCGCGGGATCTCGCACCGGATCCGGATCGT",
        "25 nmole DNA Plate Oligo",
        "Standard Desalting",
        "Full Yield",
        "100",
        "100",
        "100",
        "",
        ""
      ],
      [
        "design01 oligo plate 1",
        "A03",
        "example2_20221026-215035_1",
        "TTCGCGATCCGGACGATCCGGATCCGGTGCGAGATCC",
        "25 nmole DNA Plate Oligo",
        "Standard Desalting",
        "Full Yield",
        "100",
        "100",
        "100",
        "",
        ""
      ]
    ],
    "Oligo Plate Map": [
      [
        "Seq ID",
        "Source Plate",
        "Source Well",
        "Destination Plate",
        "Destination Well"
      ],
      [
        "example1",
        "design01 oligo plate 1",
        "A01",
        "final plate 1",
        "A01"
      ],
      [
        "example2",
        "design01 oligo plate 1",
        "A02",
        "final plate 1",
        "A02"
      ],
      [
        "example2",
        "design01 oligo plate 1",
        "A03",
        "final plate 1",
        "A02"
      ]
    ],
    "Primer Order Form": [
      [
        "Plate Name",
        "Well Position",
        "Sequence Name",
        "Sequence",
        "Scale",
        "Purification",
        "Normalization Style",
        "Quantity (nmoles)",
        "Concentration (µM)",
        "Volume (µL)",
        "Buffer",
        "Notes"
      ],
      [
        "design01 primer plate 1",
        "A01",
        "example1_20221026-215035_fwd",
        "ATGGGGACCGGATCCGGATCG",
        "25 nmole DNA Plate Oligo",
        "Standard Desalting",
        "Full Yield",
        "100",
        "100",
        "100",
        "",
        ""
      ],
      [
        "design01 primer plate 1",
        "A02",
        "example1_20221026-215035_rev",
        "TTTCGCTTTCCGATCCGGATCCG",
        "25 nmole DNA Plate Oligo",
        "Standard Desalting",
        "Full Yield",
        "100",
        "100",
        "100",
        "",
        ""
      ],
      [
        "design01 primer plate 1",
        "A03",
        "example2_20221026-215035_fwd",
        "ATGCCGGACGGATCCGGATC",
        "25 nmole DNA Plate Oligo",
        "Standard Desalting",
        "Full Yield",
        "100",
        "100",
        "100",
        "",
        ""
      ],
      [
        "design01 primer plate 1",
        "A04",
        "example2_20221026-215035_rev",
        "TTCGCGATCCGGACGATCCG",
        "25 nmole DNA Plate Oligo",
        "Standard Desalting",
        "Full Yield",
        "100",
        "100",
        "100",
        "",
        ""
      ]
    ],
    "Primer Plate Map": [
      [
        "Seq ID",
        "Source Plate",
        "Source Well",
        "Destination Plate",
        "Destination Well"
      ],
      [
        "example1",
        "design01 primer plate 1",
        "A01",
        "final plate 1",
        "A01"
      ],
      [
        "example1",
        "design01 primer plate 1",
        "A02",
        "final plate 1",
        "A01"
      ],
      [
        "example2",
        "design01 primer plate 1",
        "A03",
        "final plate 1",
        "A02"
      ],
      [
        "example2",
        "design01 primer plate 1",
        "A04",
        "final plate 1",
        "A02"
      ]
    ],
    "Oligo Info": [
      [
        "Seq Id",
        "Oligo Id",
        "Recycled",
        "Cut Position",
        "Strand",
        "Source Plate Well",
        "Destination Plate Wells",
        "Sequence",
        "Sequence Length"
      ],
      [
        "example1",
        "example1_20221026-215035_0",
        "False",
        "(0, 56)",
        "1",
        "(1, 'A01')",
        "[(1, 'A01')]",
        "ATGGGGACCGGATCCGGATCGGGATCTCGCGCCCGGATCCGGATCGGAAAGCGAAA\n",
        57
      ],
      [
        "example2",
        "example2_20221026-215035_0",
        "False",
        "(0, 50)",
        "1",
        "(1, 'A02')",
        "[(1, 'A02')]",
        "ATGCCGGACGGATCCGGATCTCGCGGGATCTCGCACCGGATCCGGATCGT\n",
        51
      ],
      [
        "example2",
        "example2_20221026-215035_1",
        "False",
        "(25, 62)",
        "-1",
        "(1, 'A03')",
        "[(1, 'A02')]",
        "TTCGCGATCCGGACGATCCGGATCCGGTGCGAGATCC\n",
        38
      ]
    ]
  },
  "Design URL": "https://storage.googleapis.com/0446c3ad-f0af-481f-9c9f-68ccdf8370cc/OligoDesigns/job_b2816ba3-3390-437b-a37c-03fc1238db94.zip?X-Goog-Algorithm=GOOG4-RSA-SHA256&X-Goog-Credential=ninthbio%40ninthbio.iam.gserviceaccount.com%2F20221026%2Fauto%2Fstorage%2Fgoog4_request&X-Goog-Date=20221026T215036Z&X-Goog-Expires=604800&X-Goog-SignedHeaders=host&X-Goog-Signature=1aea2de5420ddde3984906a2f9f3d7afc375b3a08d3bc5f170ca94dbe987e42e8ec2de5d13eee81d1391de00293f76de9f2ae60cb7ca53a9d96f87ce98059d8a312a1c1bd8bed621330529347c18ce91b1a7d76d5d8d81c0c0785119bab8ecdbc1c07d72286d6c816bd256430425ab93bc821483986ceaaf8af0cc2089c1a60f7c5b80114c444ee5e635c94649a635a2afc1a21073e68f813ea0fe73de23096a1b509bcf26be7339789f7b0fab764af7b7f3f8f9060e676c9e175845f2877db82e20209ddc42c60d57a3f0dfeb9099aa52eafe12812c8e86f37559d201532dff66cf268670a65b332a0aaac6b71e6b280a6d2470aee264b99642eebb927991dd",
  "Design Info": {
    "Total sequences": 2,
    "Total pre-recycle oligos": 3,
    "Total post-recycle oligos": 3,
    "Total pre-recycle bases": 143,
    "Total post-recycle bases": 143,
    "Total bases recycled": 0,
    "Recycle efficiency": 0,
    "Design Errors": []
  },
  "Design Errors": [],
  "Design SVG": "<?xml version=\"1.0\" encoding=\"utf-8\" ?>\n<svg baseProfile=\"full\" height=\"336.0\" version=\"1.1\" width=\"970\" xmlns=\"http://www.w3.org/2000/svg\" xmlns:ev=\"http://www.w3.org/2001/xml-events\" xmlns:xlink=\"http://www.w3.org/1999/xlink\"><defs /><g style=\"font-size:16px;font-family:Courier;font-face:bold;\"><text fill=\"rgb(0,0,0)\" style=\"white-space: pre\" textLength=\"80\" x=\"0\" y=\"16.0\">example1</text><text fill=\"rgb(0,0,0)\" style=\"white-space: pre\" textLength=\"560\" x=\"300\" y=\"16.0\">ATGGGGACCGGATCCGGATCGGGATCTCGCGCCCGGATCCGGATCGGAAAGCGAAA</text><text fill=\"rgb(0,0,0)\" style=\"white-space: pre\" textLength=\"570\" x=\"300\" y=\"32.0\">1        10        20        30        40        50      </text><text fill=\"rgb(165,33,243)\" style=\"white-space: pre\" textLength=\"560\" x=\"300\" y=\"72.0\">ATGGGGACCGGATCCGGATCGGGATCTCGCGCCCGGATCCGGATCGGAAAGCGAAA</text><text fill=\"rgb(165,33,243)\" style=\"white-space: pre\" textLength=\"260\" x=\"450.0\" y=\"120.0\">example1_20221026-215035_0</text><text fill=\"rgb(0,0,0)\" style=\"white-space: pre\" textLength=\"80\" x=\"0\" y=\"184.0\">example2</text><text fill=\"rgb(0,0,0)\" style=\"white-space: pre\" textLength=\"620\" x=\"300\" y=\"184.0\">ATGCCGGACGGATCCGGATCTCGCGGGATCTCGCACCGGATCCGGATCGTCCGGATCGCGAA</text><text fill=\"rgb(0,0,0)\" style=\"white-space: pre\" textLength=\"630\" x=\"300\" y=\"200.0\">1        10        20        30        40        50        60  </text><text fill=\"rgb(185,212,196)\" style=\"white-space: pre\" textLength=\"500\" x=\"300\" y=\"240.0\">ATGCCGGACGGATCCGGATCTCGCGGGATCTCGCACCGGATCCGGATCGT</text><text fill=\"rgb(185,212,196)\" style=\"white-space: pre\" textLength=\"260\" x=\"420.0\" y=\"288.0\">example2_20221026-215035_0</text><text fill=\"rgb(131,184,224)\" style=\"white-space: pre\" textLength=\"370\" x=\"550\" y=\"264.0\">CCTAGAGCGTGGCCTAGGCCTAGCAGGCCTAGCGCTT</text><text fill=\"rgb(131,184,224)\" style=\"white-space: pre\" textLength=\"260\" x=\"605.0\" y=\"312.0\">example2_20221026-215035_1</text></g></svg>",
  "Other": {
    "emails": "your@email.com",
    "Job Id": "b2816ba3-3390-437b-a37c-03fc1238db94"

  },
  "Finished": "b2816ba3-3390-437b-a37c-03fc1238db94"
}
```


## Sequence Complexity
The Sequence Complexity bioinformatics software package and API provide a simple and intuitive method to analyze DNA sequences for regions that may make a given sequence more difficult to synthesize. This package looks for repeat regions of various sorts, high and low GC content regions, and most importantly performs a simulated analysis of a given oligo design based on the Gibbs Free Energy (ΔG) of the pool of oligos. An example output from the Sequence Complexity package can be seen below. The sequence shown has two large interspersed repeats (dark green), high (red) and low (blue) GC regions, and a large palindromic/hairpin region (yellow). The lighter green segments (disjoint oligo pairs) at the bottom represent oligos in a design that have an abnormally high affinity (more negative ΔG), yet were not intended to anneal. This would likely result in truncated products during assembly. There is also a dark red segment (self oligo pairs) in the region of the hairpin. This oligo is likely to fold on itself and hence not be available for the assembly, resulting again in truncated product. Using this information, and the Oligo Design tool, one should be able to create a design that minimizes these synthesis issues. 

![Sequence Complexity Example](https://user-images.githubusercontent.com/2830915/198150261-7c6844a1-d53e-4712-85ef-1029701144a9.png)

### Parameters
The API accepts a JSON object containing the following:

#### API
A required string specifying which API is being requested. Valid strings are: "oligo_design", "sequence_complexity", "sequence_design"

#### Required Parameters
**required_parameters**: A required JSON object containing all three of the following entries:

###### Sequences
**sequences**: A list of JSON objects of sequence records that include the following:

1. **sequence_name**: String, a unique name for each sequence.
2. **sequence**: String, ambiguous/unambiguous DNA sequence.

###### Sequence Complexity Parameters
**sequence_complexity_parameters**: A JSON object containing the following:

The following Oligo Design parameters are used in the ΔG simulation and should represent a potential Oligo Design for assembly. 
1. **minimum_length**: Integer, default 30
The minimum size in base pairs allowed for any oligo in the design. 
2. **target_length**: Integer, default 50 
The target size in base pairs for an oligo to start with during design. 
3. **maximum_length**: Integer, default 60 
The maximum size in base pairs allowed for any oligo in the design. 
4. **minimum_overlap**: Integer, default 15
The minimum number of basepairs allowed for any given overlap between 3’ and 5’ paired oligos.
5. **maximum_overlap**: Integer, default 25
The maximum number of basepairs allowed for any given overlap between 3’ and 5’ paired oligos.
6. **minimum_zscore_cutoff**: Float, default -4.0
This is used to identify outlier oligo pairings when estimating the ΔG for each oligo pair. Lower (more negative) values will result in less overall complex regions identified. 
7. **temp**: Float, default 60.0
This is used in the ΔG calculations and should correspond to the target PCA temperature (degrees Celsius). 

### Output
An object that contains ___ in JSON. See below for an example of the JSON output.

### Error Handling
If the complexity parameters are invalid or result in an invalid complexity, verbose errors are returned for resolution.

### Use Cases
#### Homology Path
- Sequence Complexity is fully implemented in the Ninth Bio Homology Path software suite
#### RESTful Web Service
- Secure network interface
- Could be on the internet or private network
- Could take JSON as input and output
- Could interact with customer databases
#### Command Line
- Same advantages as above, but additionally can run on a network isolated computer


### Technical Examples

Below is an example of a command line to POST a job request to the Sequence Complexity API.

```
curl -0 -X POST "https://homologypath.com/router/" \
-H "Authorization: Token <YOUR_API_TOKEN_HERE>" \
-H "Content-Type: application/json" \
-d \
'
{
    "api":"sequence_complexity",
    "required_parameters":{
       "sequences":[
          {
             "sequence_name":"example1",
             "sequence":"ATGGGGACCGGATCCGGATCGGGATCTCGCGCCCGGATCCGGATCGGAAAGCGAAA"
          },
          {
             "sequence_name":"example2",
             "sequence":"ATGCCGGACGGATCCGGATCTCGCGGGATCTCGCACCGGATCCGGATCGTCCGGATCGCGAA"
          }
       ],
       "sequence_complexity_parameters":{
          "minimum_length":"25",
          "target_length":"30",
          "maximum_length":"40",
          "minimum_overlap":"15",
          "maximum_overlap":"25",
          "minimum_zscore_cutoff":"-4.0",
          "temp":"60"
       }
    }
}'
