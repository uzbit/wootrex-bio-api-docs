# wootrex bio API Documentation

## Oligo Design

The Oligo Design bioinformatics software package and API provide a simple and intuitive method to create single stranded oligonucleotide sequences that, when pooled together, assemble into the requested double stranded DNA molecule. It also provides an ability to take a library of sequences with homologous and variable regions and create a design across the entire library that minimizes the number of homologous region oligos, while also preserving the required variable region oligos. Primers for amplification are provided and recycled where possible. Both the oligo design and the primer design are presented with their respective plate maps for ease of automation in the wet lab. The output is easily consumed by all major oligo manufacturers such as IDT, Twist, etc. 

### Parameters
The API accepts a JSON object containing the following:

#### API
**api**: A required string specifying which API is being requested. Valid strings are: "oligo_design", "sequence_complexity", "sequence_design"

#### Required Parameters
**required_parameters**: A required JSON object containing the following entries:

###### Sequences
- **sequences**: A list of JSON objects of sequence records that include the following:

    - **sequence_name**: String, a unique name for each sequence.
    - **sequence**: String, ambiguous/unambiguous DNA sequence.
    - **primer_length**: Integer, number of base pairs for the amplification primer.

###### Oligo Design Parameters
- **oligo_design_parameters**: A JSON object containing the following:

    - **minimum_length**: Integer, default 30
The minimum size in base pairs allowed for any oligo in the design. 
    - **target_length**: Integer, default 50 
The target size in base pairs for an oligo to start with during design. 
    - **maximum_length**: Integer, default 60 
The maximum size in base pairs allowed for any oligo in the design. 
    - **minimum_overlap**: Integer, default 15
The minimum number of basepairs allowed for any given overlap between 3??? and 5??? paired oligos.
    - **maximum_overlap**: Integer, default 25
The maximum number of basepairs allowed for any given overlap between 3??? and 5??? paired oligos.

###### Primer Design Parameters 
- **primer_design_parameters**: A JSON object containing the following:

    - **target_primer_length**: Integer, default 20
The minimum size in base pairs allowed for any oligo in the design. 
    - **tm_optimize_primers**: Boolean, default True
Wether or not to melting temperature (TM) optimize the amplification primers. 
    - **target_primer_tm_temp**: Integer, default 60
The temperature (in degrees Celsius) at wich PCA is performed. This is only used if **tm_optimize_primers** is **True**


#### Optional Parameters
- **optional_parameters**: An optional JSON object containing any of the following:

    - **design_type**: String, default "maximize_recycling".
Specifies the type of Oligo Design to perform. Valid types are: "maximize_recycling", "minimize_complexity", "gapped_design", "standard"
    - **design_name**: String, default "design01"
User specified name for the design. The files emailed will have this name.
    - **recycle_oligos**: Boolean, default True.
Perform oligo recycling across the entire set of sequences provided in Sequence Records.
    - **source_plate_size**: Integer, default 96.
Number of wells of the oligo plate to order from the oligo manufacturer. Currently must be either 96 or 384. 
    - **destination_plate_size**: Integer, default 96.
Number of wells of the pooling plate. Currently must be either 96 or 384. 
    - **row_major**: Boolean, default True
Provide the list of oligos in row first (A1, A2, A3, etc) order if True, otherwise provide this list in column first order (A1, B1, C1, etc).
    - **emails**: String, default is empty
A comma separated list of emails to which to send the design files.
    - **partition_identity**: Float, default 0.95
Used only for Design Type = "maximize_recycling". This is used to help inform the Oligo Design about how best to partition the sequences for maximizing the recycling. Tuning this parameter will affect the overall recycle efficiency. This should be a number between 0.0 and 1.0.
    - **min_zscore_cutoff**: Float, default -4.0
Used only for Design Type = "minimize_complexity". This is used to help inform the Oligo Design how to identify outlier oligo pairings when estimating the ??G for each oligo pair. Lower (more negative) values will result in less overall complex regions identified. 
    - **temp**: Float, default 60.0
Used only for Design Type = "minimize_complexity". This is used in the ??G calculations and should correspond to the target PCA temperature (degrees Celsius). 
    - **optimize_overlap_tm**: Boolean, default False (NOT IMPLEMENTED AT API LEVEL YET)
Make oligo overlaps TM optimized to **overlap_tm_target_temp**
    - **overlap_tm_target_temp**: Float, default 60.0 (NOT IMPLEMENTED AT API LEVEL YET)
This should correspond to the target PCA temperature (degrees Celsius). 

### Output
An object that contains the oligo and primer designs, plate maps, and visualization  in JSON. See below for an example of the JSON output.

### Error Handling
If the design parameters are invalid or result in an invalid design (low success of assembly, or doesn???t match parameters) verbose errors are returned for resolution.

### API Request Examples

Below is an example of a command line to POST a job request to the Oligo Design API.

```
curl -0 -X POST "https://homologypath.com/router/" \
-H "Authorization: Token YOUR_API_TOKEN_HERE" \
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
-H "Authorization: Token YOUR_API_TOKEN_HERE" \
-H "Content-Type: application/json" \
-d \
'{"api": "oligo_design", "job_id":"b2816ba3-3390-437b-a37c-03fc1238db94"}'
```

Depending on how large the request was, this polling request may need to be done periodically until the calculation is complete. Once the design is complete, there will be a `Finished` field and the response should look similar to the following:

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
        "Concentration (??M)",
        "Volume (??L)",
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
        "Concentration (??M)",
        "Volume (??L)",
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
The Sequence Complexity bioinformatics software package and API provide a simple and intuitive method to analyze DNA sequences for regions that may make a given sequence more difficult to synthesize. This package looks for repeat regions of various sorts, high and low GC content regions, and most importantly performs a simulated analysis of a given oligo design based on the Gibbs Free Energy (??G) of the pool of oligos for a given sequence. An example output from the Sequence Complexity package can be seen below. The sequence shown has two large interspersed repeats (light green), high (light red) and low (light blue) GC regions, and a large palindromic/hairpin region (yellow). The lighter green segments (disjoint oligo pairs) at the bottom represent oligos in a design that have an abnormally high affinity (more negative ??G), yet were not intended to anneal. This would likely result in truncated products during assembly. There is also a dark red segment (self oligo pairs) in the region of the hairpin. This oligo is likely to fold on itself and hence not be available for the assembly, resulting again in truncated product. Using this information, and the Oligo Design tool, one should be able to create a design that minimizes these synthesis issues. 

![Sequence Complexity Example](https://user-images.githubusercontent.com/2830915/207479837-17bea843-9a36-4dbd-905b-79120fe5bf03.png)

### Parameters
The API accepts a JSON object containing the following:

#### API
**api**: A required string specifying which API is being requested. Valid strings are: "oligo_design", "sequence_complexity", "sequence_design"

#### Required Parameters
**required_parameters**: A required JSON object containing the following entries:

###### Sequences
- **sequences**: A list of JSON objects of sequence records that include the following:

    - **sequence_name**: String, a unique name for each sequence.
    - **sequence**: String, ambiguous/unambiguous DNA sequence.

###### Sequence Complexity Parameters
- **sequence_complexity_parameters**: A JSON object containing the following: 
    - **minimum_length**: Integer, default 30
The minimum size in base pairs allowed for any oligo in the design. 
    - **target_length**: Integer, default 50 
The target size in base pairs for an oligo to start with during design. 
    - **maximum_length**: Integer, default 60 
The maximum size in base pairs allowed for any oligo in the design. 
    - **minimum_overlap**: Integer, default 15
The minimum number of basepairs allowed for any given overlap between 3??? and 5??? paired oligos.
    - **maximum_overlap**: Integer, default 25
The maximum number of basepairs allowed for any given overlap between 3??? and 5??? paired oligos.
    - **minimum_zscore_cutoff**: Float, default -4.0
This is used to identify outlier oligo pairings when estimating the ??G for each oligo pair. Lower (more negative) values will result in **less** overall complex regions identified. 
    - **temp**: Float, default 60.0
This is used in the ??G calculations and should correspond to the target PCA temperature (degrees Celsius). 

### Output
An object that contains complexity results in JSON format. See below for an example of the JSON output.


#### Complexity Result Definitions
A complexity result will be one of the following types:

##### High GC Region (Code 101)
A high GC region is identified as non-overlapping 40bp windows of the sequence with at least 70% GC content. The "value" field returned by the API is the ratio of GC in the 40bp window. Represented in light red in the result image.

##### Low GC Region (Code 102)
A low GC region is identified as non-overlapping 40bp windows of the sequence with at most 30% GC content. The "value" field returned by the API is the ratio of GC in the 40bp window. Represented in light blue in the result image.

##### Interspersed Repeat (Code 201)
An interspersed repeat is identified as at least two non-contiguous parts of the sequence of at least 7bp in length. Both forward and reverse strands are compared. The "value" field returned by the API is a BLASTn bitscore for each pair. Represented in dark green in the result image.

##### Palindrome Repeat (Code 202)
A palindrome repeat is identified as a contiguous region that would likely form a hairpin. These must be at least 10bp and of at least 85% identity. The "value" field returned by the API is a score that is a combination of length and identity of the palindrome. Represented in dark green in the result image. Represented in yellow in the result image. 

##### ??G - Overlapping Oligo Pairs (Code 401)
Two oligos designed to overlap but have a higher (more positive) the other oligos do that are intended to overlap by at least abs(Minimum Z-Score Cutoff). Represented in darker blue in the result image. 

##### ??G - Self Oligo Pairs (Code 402)
An oligo that has higher affinity to itself than the other oligos do to themselves by at least Minimum Z-Score Cutoff. Represented in darker red in the result image. 

##### ??G - Disjoint Oligo Pairs (Code 403)
An oligo pair that have higher (more negative ??G) affinity to each other than other oligos do by at least Minimum Z-Score Cutoff. Represented in darker green in the result image. 

##### ??G - Z-Score Definition
The Z-score is calculated based on a simulation of a distribution of designed sequences of identical design and similar GC content.


### Error Handling
If the complexity parameters are invalid or result in an invalid complexity, verbose errors are returned for resolution.

### API Request Examples

Below is an example of a command line to POST a job request to the Sequence Complexity API.

```
curl -0 -X POST "https://homologypath.com/router/" \
-H "Authorization: Token YOUR_API_TOKEN_HERE" \
-H "Content-Type: application/json" \
-d \
'
{
  "api": "sequence_complexity",
  "required_parameters": {
    "sequences": [
      {
        "sequence_name": "example1",
        "sequence": "ATGGGGACCGGATCCGGATCGGGTTTCATAACTATACTCGTAAGGATCATGTTATTGATTTCTTCAAAGGTTACTGTGGGTCTCCGGGCCCCCCCGTCCACCCAGACCGACCGAACTCTCCCGTTTAGGACCCTAGTAAGTCATCATTGGTATATGAATGCGACCCCGAAGAACTGTGGGTCTCCGGGCCCCCCCGTCCACCCAGACCGACCGAACTCTCCCGCAATTCTATAAGAATGCACACTGCATCGATACATAAAACGTCTCGATCGCGCCGGGAAAGGTACGCACGCGGTATATACCGCGTGCGTACCTTTCCCGGCTCCTTCCAGAGGTATGTGGCTGCGTGGTCAAAAGTGCGGCATTCGTATTTGCTCCTCGTGTTTACTCTCACAAACTTGACCTGGAGATCAAGGAGATGCTTCTTGTGGAACTGGACAACGCAATCTCGCGCCCGGATCCGGATCGGAAAGCGAAA"
      },
      {
        "sequence_name": "example2",
        "sequence": "ATGCCGGACGGATCCGGATCTCGCGGGATCTCGCACCGGATCCGGATCTTTCATAACTATACTCGTAAGGATCATGTTATTGATTTCTTCAAAGGTTACTGTGGGTCTCCGGGCCCCCCCGTCCACCCAGACCGACCGAACTCTCCCGTTTAGGACCCTAGTAAGTCATCATTGGTATATGAATGCGACCCCGAAGAACTGTGGGTCTCCGGGCCCCCCCGTCCACCCAGACCGACCGAACTCTCCCGCAATTCTATAAGAATGCACACTGCATCGATACATAAAACGTCTCGATCGCGCCGGGAAAGGTACGCACGCGGTATATACCGCGTGCGTACCTTTCCCGGCTCCTTCCAGAGGTATGTGGCTGCGTGGTCAAAAGTGCGGCATTCGTATTTGCTCCTCGTGTTTACTCTCACAAACTTGACCTGGAGATCAAGGAGATGCTTCTTGTGGAACTGGACAACGCAGTCCGGATCGCGAA"
      }
    ],
    "sequence_complexity_parameters": {
      "minimum_length": "25",
      "target_length": "30",
      "maximum_length": "40",
      "minimum_overlap": "15",
      "maximum_overlap": "25",
      "minimum_zscore_cutoff": "-4.0",
      "temp": "60"
    }
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
-H "Authorization: Token YOUR_API_TOKEN_HERE" \
-H "Content-Type: application/json" \
-d \
'{"api": "sequence_complexity", "job_id":"b2816ba3-3390-437b-a37c-03fc1238db94"}'
```

Depending on how large the request was, this polling request may need to be done periodically until the calculation is complete. Once the complexity calculation is complete, there will be a `Finished` field and the response should look similar to the following:


```
{
   "Complexity":{
      "example1":[
         {
            "message":"interspersed repeat at (72:123) - (172:223)",
            "code":201,
            "value":1,
            "details":[
               {
                  "position":"(72:123)",
                  "value":95.3
               },
               {
                  "position":"(172:223)",
                  "value":95.3
               }
            ]
         },
         {
            "message":"interspersed repeat at (7:22) - (454:469)",
            "code":201,
            "value":1,
            "details":[
               {
                  "position":"(7:22)",
                  "value":28.8
               },
               {
                  "position":"(454:469)",
                  "value":28.8
               }
            ]
         },
         {
            "message":"interspersed repeat at (269:280) - (448:459)",
            "code":201,
            "value":1,
            "details":[
               {
                  "position":"(269:280)",
                  "value":15.9
               },
               {
                  "position":"(448:459)",
                  "value":15.9
               }
            ]
         },
         {
            "message":"palindromic repeat at (259:298) - (298:337)",
            "code":202,
            "value":1,
            "details":[
               {
                  "position":"(259:298)",
                  "value":78
               },
               {
                  "position":"(298:337)",
                  "value":78
               }
            ]
         },
         {
            "message":"Low GC at (24:64)",
            "code":102,
            "value":1,
            "details":[
               {
                  "position":"(24:64)",
                  "value":0.29749999999999993
               }
            ]
         },
         {
            "message":"High GC at (68:108), High GC at (158:198)",
            "code":101,
            "value":1,
            "details":[
               {
                  "position":"(68:108)",
                  "value":0.7024999999999999
               },
               {
                  "position":"(158:198)",
                  "value":0.7074999999999998
               }
            ]
         },
         {
            "message":"Total sequence GC 53.77%",
            "code":103,
            "value":0,
            "details":[
               {
                  "position":"(0:478)",
                  "value":0.5376569037656904
               }
            ]
         },
         {
            "message":"Self oligo pair at (279:310) - (279:310) is an outlier.",
            "code":403,
            "value":1,
            "details":[
               {
                  "positionA":"(279:310)",
                  "positionB":"(279:310)",
                  "deltaG":-33.22,
                  "zScore":-5.9387986158125585
               }
            ]
         },
         {
            "message":"Disjoint oligo pair at (0:31) - (449:478) is an outlier.",
            "code":402,
            "value":1,
            "details":[
               {
                  "positionA":"(0:31)",
                  "positionB":"(449:478)",
                  "deltaG":-21.75,
                  "zScore":-5.49545251642264
               }
            ]
         },
         {
            "message":"Disjoint oligo pair at (62:93) - (170:201) is an outlier.",
            "code":402,
            "value":1,
            "details":[
               {
                  "positionA":"(62:93)",
                  "positionB":"(170:201)",
                  "deltaG":-37.16,
                  "zScore":-10.671630477489774
               }
            ]
         },
         {
            "message":"Disjoint oligo pair at (77:108) - (170:201) is an outlier.",
            "code":402,
            "value":1,
            "details":[
               {
                  "positionA":"(77:108)",
                  "positionB":"(170:201)",
                  "deltaG":-17.39,
                  "zScore":-4.030940127697609
               }
            ]
         },
         {
            "message":"Disjoint oligo pair at (77:108) - (186:217) is an outlier.",
            "code":402,
            "value":1,
            "details":[
               {
                  "positionA":"(77:108)",
                  "positionB":"(186:217)",
                  "deltaG":-42.53,
                  "zScore":-12.47539917644698
               }
            ]
         },
         {
            "message":"Disjoint oligo pair at (93:124) - (201:232) is an outlier.",
            "code":402,
            "value":1,
            "details":[
               {
                  "positionA":"(93:124)",
                  "positionB":"(201:232)",
                  "deltaG":-35.1,
                  "zScore":-9.979681963550885
               }
            ]
         },
         {
            "message":"Disjoint oligo pair at (263:294) - (294:325) is an outlier.",
            "code":402,
            "value":1,
            "details":[
               {
                  "positionA":"(263:294)",
                  "positionB":"(294:325)",
                  "deltaG":-31.41,
                  "zScore":-8.740220790524425
               }
            ]
         }
      ],
      "example2":[
         {
            "message":"interspersed repeat at (97:148) - (197:248)",
            "code":201,
            "value":1,
            "details":[
               {
                  "position":"(97:148)",
                  "value":95.3
               },
               {
                  "position":"(197:248)",
                  "value":95.3
               }
            ]
         },
         {
            "message":"interspersed repeat at (8:21) - (10:18) - (36:49) - (471:479)",
            "code":201,
            "value":1,
            "details":[
               {
                  "position":"(8:21)",
                  "value":25.1
               },
               {
                  "position":"(10:18)",
                  "value":15.9
               },
               {
                  "position":"(36:49)",
                  "value":25.1
               },
               {
                  "position":"(471:479)",
                  "value":15.9
               }
            ]
         },
         {
            "message":"palindromic repeat at (284:323) - (323:362)",
            "code":202,
            "value":1,
            "details":[
               {
                  "position":"(284:323)",
                  "value":78
               },
               {
                  "position":"(323:362)",
                  "value":78
               }
            ]
         },
         {
            "message":"High GC at (1:41), High GC at (93:133), High GC at (183:223)",
            "code":101,
            "value":1,
            "details":[
               {
                  "position":"(1:41)",
                  "value":0.7049999999999998
               },
               {
                  "position":"(93:133)",
                  "value":0.7024999999999999
               },
               {
                  "position":"(183:223)",
                  "value":0.7074999999999998
               }
            ]
         },
         {
            "message":"Low GC at (47:87)",
            "code":102,
            "value":1,
            "details":[
               {
                  "position":"(47:87)",
                  "value":0.2975
               }
            ]
         },
         {
            "message":"Total sequence GC 54.13%",
            "code":103,
            "value":0,
            "details":[
               {
                  "position":"(0:484)",
                  "value":0.5413223140495868
               }
            ]
         },
         {
            "message":"Self oligo pair at (310:341) - (310:341) is an outlier.",
            "code":403,
            "value":1,
            "details":[
               {
                  "positionA":"(310:341)",
                  "positionB":"(310:341)",
                  "deltaG":-36.36,
                  "zScore":-6.5054610749961395
               }
            ]
         },
         {
            "message":"Disjoint oligo pair at (0:31) - (201:232) is an outlier.",
            "code":402,
            "value":1,
            "details":[
               {
                  "positionA":"(0:31)",
                  "positionB":"(201:232)",
                  "deltaG":-18.79,
                  "zScore":-4.333570082517523
               }
            ]
         },
         {
            "message":"Disjoint oligo pair at (93:124) - (201:232) is an outlier.",
            "code":402,
            "value":1,
            "details":[
               {
                  "positionA":"(93:124)",
                  "positionB":"(201:232)",
                  "deltaG":-42.91,
                  "zScore":-12.234944848030633
               }
            ]
         },
         {
            "message":"Disjoint oligo pair at (108:139) - (186:217) is an outlier.",
            "code":402,
            "value":1,
            "details":[
               {
                  "positionA":"(108:139)",
                  "positionB":"(186:217)",
                  "deltaG":-23.14,
                  "zScore":-5.758569262865037
               }
            ]
         },
         {
            "message":"Disjoint oligo pair at (108:139) - (217:248) is an outlier.",
            "code":402,
            "value":1,
            "details":[
               {
                  "positionA":"(108:139)",
                  "positionB":"(217:248)",
                  "deltaG":-35.92,
                  "zScore":-9.945118578920491
               }
            ]
         },
         {
            "message":"Disjoint oligo pair at (124:155) - (232:263) is an outlier.",
            "code":402,
            "value":1,
            "details":[
               {
                  "positionA":"(124:155)",
                  "positionB":"(232:263)",
                  "deltaG":-22.62,
                  "zScore":-5.588224533260277
               }
            ]
         },
         {
            "message":"Disjoint oligo pair at (294:325) - (325:356) is an outlier.",
            "code":402,
            "value":1,
            "details":[
               {
                  "positionA":"(294:325)",
                  "positionB":"(325:356)",
                  "deltaG":-35.19,
                  "zScore":-9.705980785436884
               }
            ]
         }
      ]
   },
   "Complexity URL":"https://storage.googleapis.com/0446c3ad-f0af-481f-9c9f-68ccdf8370cc/SequenceComplexitys/job_b0739602-a964-40f8-8795-68d946e24d09.zip?X-Goog-Algorithm=GOOG4-RSA-SHA256&X-Goog-Credential=ninthbio%40ninthbio.iam.gserviceaccount.com%2F20221103%2Fauto%2Fstorage%2Fgoog4_request&X-Goog-Date=20221103T021444Z&X-Goog-Expires=604800&X-Goog-SignedHeaders=host&X-Goog-Signature=1a005ac6559e1871a7ece5e429a7a2e0a49afd5b1c62a96d94dc9c6b0fdca47f0d87868d3a1c2fd30347ae271dbc6ef04807db30c1fde9d45b4806c5776c148c9a264907c5978cc487ba47321fc4ad23c08394bc777685e696238e2c787b5cdd3371cbc5ac05a7a56ad609705858b52ecacc89c170f3a1d25c9bf2761ea6801f9457a3a7f520ef6332fccd8bae9489bd025dbb83b30f46ec1054e30962153a4b1f0994ba0ac691104e0b676caabfeac2a29391c43b9084604c3b8b54760df41d708bba0e6ce1e19337877e30051923646a7714cf4c85bad6e6fba13ab7e18671aa673a2db7b3583a96b7585be4c94f8c592a373020f02af59c6607347fc35cc3",
   "Complexity Info":"Sequence Complexity for example1:<br>Complexity Code 201:<br>interspersed repeat at (72:123) - (172:223)<br>Repeat Info:<br>(72:123):&emsp;95.3<br>(172:223):&emsp;95.3<p></p>Complexity Code 201:<br>interspersed repeat at (7:22) - (454:469)<br>Repeat Info:<br>(7:22):&emsp;28.8<br>(454:469):&emsp;28.8<p></p>Complexity Code 201:<br>interspersed repeat at (269:280) - (448:459)<br>Repeat Info:<br>(269:280):&emsp;15.9<br>(448:459):&emsp;15.9<p></p>Complexity Code 202:<br>palindromic repeat at (259:298) - (298:337)<br>Repeat Info:<br>(259:298):&emsp;78<br>(298:337):&emsp;78<p></p>Complexity Code 102:<br>Low GC at (24:64)<br>GC Info:<br>(24:64):&emsp;29.75% GC<p></p>Complexity Code 101:<br>High GC at (68:108), High GC at (158:198)<br>GC Info:<br>(68:108):&emsp;70.25% GC<br>(158:198):&emsp;70.75% GC<p></p>Complexity Code 103:<br>Total sequence GC 53.77%<br>GC Info:<br>(0:478):&emsp;53.77% GC<p></p>Complexity Code 403:<br>Self oligo pair at (279:310) - (279:310) is an outlier.<br>Free Energy Info:<br>deltaG: -33.22 kcal/mol<br>Z-score: -5.94<p></p>Complexity Code 402:<br>Disjoint oligo pair at (0:31) - (449:478) is an outlier.<br>Free Energy Info:<br>deltaG: -21.75 kcal/mol<br>Z-score: -5.50<p></p>Complexity Code 402:<br>Disjoint oligo pair at (62:93) - (170:201) is an outlier.<br>Free Energy Info:<br>deltaG: -37.16 kcal/mol<br>Z-score: -10.67<p></p>Complexity Code 402:<br>Disjoint oligo pair at (77:108) - (170:201) is an outlier.<br>Free Energy Info:<br>deltaG: -17.39 kcal/mol<br>Z-score: -4.03<p></p>Complexity Code 402:<br>Disjoint oligo pair at (77:108) - (186:217) is an outlier.<br>Free Energy Info:<br>deltaG: -42.53 kcal/mol<br>Z-score: -12.48<p></p>Complexity Code 402:<br>Disjoint oligo pair at (93:124) - (201:232) is an outlier.<br>Free Energy Info:<br>deltaG: -35.10 kcal/mol<br>Z-score: -9.98<p></p>Complexity Code 402:<br>Disjoint oligo pair at (263:294) - (294:325) is an outlier.<br>Free Energy Info:<br>deltaG: -31.41 kcal/mol<br>Z-score: -8.74<p></p>Sequence Complexity for example2:<br>Complexity Code 201:<br>interspersed repeat at (97:148) - (197:248)<br>Repeat Info:<br>(97:148):&emsp;95.3<br>(197:248):&emsp;95.3<p></p>Complexity Code 201:<br>interspersed repeat at (8:21) - (10:18) - (36:49) - (471:479)<br>Repeat Info:<br>(8:21):&emsp;25.1<br>(10:18):&emsp;15.9<br>(36:49):&emsp;25.1<br>(471:479):&emsp;15.9<p></p>Complexity Code 202:<br>palindromic repeat at (284:323) - (323:362)<br>Repeat Info:<br>(284:323):&emsp;78<br>(323:362):&emsp;78<p></p>Complexity Code 101:<br>High GC at (1:41), High GC at (93:133), High GC at (183:223)<br>GC Info:<br>(1:41):&emsp;70.50% GC<br>(93:133):&emsp;70.25% GC<br>(183:223):&emsp;70.75% GC<p></p>Complexity Code 102:<br>Low GC at (47:87)<br>GC Info:<br>(47:87):&emsp;29.75% GC<p></p>Complexity Code 103:<br>Total sequence GC 54.13%<br>GC Info:<br>(0:484):&emsp;54.13% GC<p></p>Complexity Code 403:<br>Self oligo pair at (310:341) - (310:341) is an outlier.<br>Free Energy Info:<br>deltaG: -36.36 kcal/mol<br>Z-score: -6.51<p></p>Complexity Code 402:<br>Disjoint oligo pair at (0:31) - (201:232) is an outlier.<br>Free Energy Info:<br>deltaG: -18.79 kcal/mol<br>Z-score: -4.33<p></p>Complexity Code 402:<br>Disjoint oligo pair at (93:124) - (201:232) is an outlier.<br>Free Energy Info:<br>deltaG: -42.91 kcal/mol<br>Z-score: -12.23<p></p>Complexity Code 402:<br>Disjoint oligo pair at (108:139) - (186:217) is an outlier.<br>Free Energy Info:<br>deltaG: -23.14 kcal/mol<br>Z-score: -5.76<p></p>Complexity Code 402:<br>Disjoint oligo pair at (108:139) - (217:248) is an outlier.<br>Free Energy Info:<br>deltaG: -35.92 kcal/mol<br>Z-score: -9.95<p></p>Complexity Code 402:<br>Disjoint oligo pair at (124:155) - (232:263) is an outlier.<br>Free Energy Info:<br>deltaG: -22.62 kcal/mol<br>Z-score: -5.59<p></p>Complexity Code 402:<br>Disjoint oligo pair at (294:325) - (325:356) is an outlier.<br>Free Energy Info:<br>deltaG: -35.19 kcal/mol<br>Z-score: -9.71<p></p>",
   "Complexity Errors":[
      
   ],
   "Complexity SVG":"<?xml version=\"1.0\" encoding=\"utf-8\" ?>\n<svg baseProfile=\"full\" height=\"840.0\" version=\"1.1\" width=\"5190\" xmlns=\"http://www.w3.org/2000/svg\" xmlns:ev=\"http://www.w3.org/2001/xml-events\" xmlns:xlink=\"http://www.w3.org/1999/xlink\"><defs /><g style=\"font-size:16px;font-family:Courier;font-face:bold;\"><text fill=\"rgb(0,0,0)\" style=\"white-space: pre\" textLength=\"80\" x=\"0\" y=\"16\">example1</text><text fill=\"rgb(0,0,0)\" style=\"white-space: pre\" textLength=\"4780\" x=\"300\" y=\"16\">ATGGGGACCGGATCCGGATCGGGTTTCATAACTATACTCGTAAGGATCATGTTATTGATTTCTTCAAAGGTTACTGTGGGTCTCCGGGCCCCCCCGTCCACCCAGACCGACCGAACTCTCCCGTTTAGGACCCTAGTAAGTCATCATTGGTATATGAATGCGACCCCGAAGAACTGTGGGTCTCCGGGCCCCCCCGTCCACCCAGACCGACCGAACTCTCCCGCAATTCTATAAGAATGCACACTGCATCGATACATAAAACGTCTCGATCGCGCCGGGAAAGGTACGCACGCGGTATATACCGCGTGCGTACCTTTCCCGGCTCCTTCCAGAGGTATGTGGCTGCGTGGTCAAAAGTGCGGCATTCGTATTTGCTCCTCGTGTTTACTCTCACAAACTTGACCTGGAGATCAAGGAGATGCTTCTTGTGGAACTGGACAACGCAATCTCGCGCCCGGATCCGGATCGGAAAGCGAAA</text><text fill=\"rgb(0,0,0)\" style=\"white-space: pre\" textLength=\"4790\" x=\"300\" y=\"32\">1        10        20        30        40        50        60        70        80        90        100       110       120       130       140       150       160       170       180       190       200       210       220       230       240       250       260       270       280       290       300       310       320       330       340       350       360       370       380       390       400       410       420       430       440       450       460       470       </text><text fill=\"rgb(100,100,100)\" style=\"white-space: pre\" textLength=\"120\" x=\"0\" y=\"72\">interspersed</text><text fill=\"rgb(200,200,200)\" style=\"white-space: pre\" textLength=\"4780\" x=\"300\" y=\"72\">----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------</text><text fill=\"rgb(63,106,115)\" style=\"white-space: pre\" textLength=\"510\" x=\"1020\" y=\"72\">?????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????</text><text fill=\"rgb(63,106,115)\" style=\"white-space: pre\" textLength=\"510\" x=\"2020\" y=\"72\">?????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????</text><text fill=\"rgb(100,100,100)\" style=\"white-space: pre\" textLength=\"120\" x=\"0\" y=\"96\">interspersed</text><text fill=\"rgb(200,200,200)\" style=\"white-space: pre\" textLength=\"4780\" x=\"300\" y=\"96\">----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------</text><text fill=\"rgb(63,106,115)\" style=\"white-space: pre\" textLength=\"150\" x=\"370\" y=\"96\">?????????????????????????????????????????????</text><text fill=\"rgb(63,106,115)\" style=\"white-space: pre\" textLength=\"150\" x=\"4840\" y=\"96\">?????????????????????????????????????????????</text><text fill=\"rgb(100,100,100)\" style=\"white-space: pre\" textLength=\"120\" x=\"0\" y=\"120\">interspersed</text><text fill=\"rgb(200,200,200)\" style=\"white-space: pre\" textLength=\"4780\" x=\"300\" y=\"120\">----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------</text><text fill=\"rgb(63,106,115)\" style=\"white-space: pre\" textLength=\"110\" x=\"2990\" y=\"120\">?????????????????????????????????</text><text fill=\"rgb(63,106,115)\" style=\"white-space: pre\" textLength=\"110\" x=\"4780\" y=\"120\">?????????????????????????????????</text><text fill=\"rgb(100,100,100)\" style=\"white-space: pre\" textLength=\"110\" x=\"0\" y=\"144\">palindromic</text><text fill=\"rgb(200,200,200)\" style=\"white-space: pre\" textLength=\"4780\" x=\"300\" y=\"144\">----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------</text><text fill=\"rgb(250,199,72)\" style=\"white-space: pre\" textLength=\"390\" x=\"2890\" y=\"144\">?????????????????????????????????????????????????????????????????????????????????????????????????????????????????????</text><text fill=\"rgb(250,199,72)\" style=\"white-space: pre\" textLength=\"390\" x=\"3280\" y=\"144\">?????????????????????????????????????????????????????????????????????????????????????????????????????????????????????</text><text fill=\"rgb(100,100,100)\" style=\"white-space: pre\" textLength=\"60\" x=\"0\" y=\"168\">low GC</text><text fill=\"rgb(200,200,200)\" style=\"white-space: pre\" textLength=\"4780\" x=\"300\" y=\"168\">----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------</text><text fill=\"rgb(91,192,255)\" style=\"white-space: pre\" textLength=\"400\" x=\"540\" y=\"168\">????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????</text><text fill=\"rgb(100,100,100)\" style=\"white-space: pre\" textLength=\"70\" x=\"0\" y=\"192\">high GC</text><text fill=\"rgb(200,200,200)\" style=\"white-space: pre\" textLength=\"4780\" x=\"300\" y=\"192\">----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------</text><text fill=\"rgb(255,66,63)\" style=\"white-space: pre\" textLength=\"400\" x=\"980\" y=\"192\">????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????</text><text fill=\"rgb(255,66,63)\" style=\"white-space: pre\" textLength=\"400\" x=\"1880\" y=\"192\">????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????</text><text fill=\"rgb(100,100,100)\" style=\"white-space: pre\" textLength=\"100\" x=\"0\" y=\"216\">self pairs</text><text fill=\"rgb(200,200,200)\" style=\"white-space: pre\" textLength=\"4780\" x=\"300\" y=\"216\">----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------</text><text fill=\"rgb(168,58,36)\" style=\"white-space: pre\" textLength=\"310\" x=\"3090\" y=\"216\">?????????????????????????????????????????????????????????????????????????????????????????????</text><text fill=\"rgb(168,58,36)\" style=\"white-space: pre\" textLength=\"310\" x=\"3090\" y=\"216\">?????????????????????????????????????????????????????????????????????????????????????????????</text><text fill=\"rgb(100,100,100)\" style=\"white-space: pre\" textLength=\"140\" x=\"0\" y=\"240\">disjoint pairs</text><text fill=\"rgb(200,200,200)\" style=\"white-space: pre\" textLength=\"4780\" x=\"300\" y=\"240\">----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------</text><text fill=\"rgb(91,146,121)\" style=\"white-space: pre\" textLength=\"310\" x=\"300\" y=\"240\">?????????????????????????????????????????????????????????????????????????????????????????????</text><text fill=\"rgb(91,146,121)\" style=\"white-space: pre\" textLength=\"290\" x=\"4790\" y=\"240\">???????????????????????????????????????????????????????????????????????????????????????</text><text fill=\"rgb(100,100,100)\" style=\"white-space: pre\" textLength=\"140\" x=\"0\" y=\"264\">disjoint pairs</text><text fill=\"rgb(200,200,200)\" style=\"white-space: pre\" textLength=\"4780\" x=\"300\" y=\"264\">----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------</text><text fill=\"rgb(91,146,121)\" style=\"white-space: pre\" textLength=\"310\" x=\"920\" y=\"264\">?????????????????????????????????????????????????????????????????????????????????????????????</text><text fill=\"rgb(91,146,121)\" style=\"white-space: pre\" textLength=\"310\" x=\"2000\" y=\"264\">?????????????????????????????????????????????????????????????????????????????????????????????</text><text fill=\"rgb(100,100,100)\" style=\"white-space: pre\" textLength=\"140\" x=\"0\" y=\"288\">disjoint pairs</text><text fill=\"rgb(200,200,200)\" style=\"white-space: pre\" textLength=\"4780\" x=\"300\" y=\"288\">----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------</text><text fill=\"rgb(91,146,121)\" style=\"white-space: pre\" textLength=\"310\" x=\"1070\" y=\"288\">?????????????????????????????????????????????????????????????????????????????????????????????</text><text fill=\"rgb(91,146,121)\" style=\"white-space: pre\" textLength=\"310\" x=\"2000\" y=\"288\">?????????????????????????????????????????????????????????????????????????????????????????????</text><text fill=\"rgb(100,100,100)\" style=\"white-space: pre\" textLength=\"140\" x=\"0\" y=\"312\">disjoint pairs</text><text fill=\"rgb(200,200,200)\" style=\"white-space: pre\" textLength=\"4780\" x=\"300\" y=\"312\">----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------</text><text fill=\"rgb(91,146,121)\" style=\"white-space: pre\" textLength=\"310\" x=\"1070\" y=\"312\">?????????????????????????????????????????????????????????????????????????????????????????????</text><text fill=\"rgb(91,146,121)\" style=\"white-space: pre\" textLength=\"310\" x=\"2160\" y=\"312\">?????????????????????????????????????????????????????????????????????????????????????????????</text><text fill=\"rgb(100,100,100)\" style=\"white-space: pre\" textLength=\"140\" x=\"0\" y=\"336\">disjoint pairs</text><text fill=\"rgb(200,200,200)\" style=\"white-space: pre\" textLength=\"4780\" x=\"300\" y=\"336\">----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------</text><text fill=\"rgb(91,146,121)\" style=\"white-space: pre\" textLength=\"310\" x=\"1230\" y=\"336\">?????????????????????????????????????????????????????????????????????????????????????????????</text><text fill=\"rgb(91,146,121)\" style=\"white-space: pre\" textLength=\"310\" x=\"2310\" y=\"336\">?????????????????????????????????????????????????????????????????????????????????????????????</text><text fill=\"rgb(100,100,100)\" style=\"white-space: pre\" textLength=\"140\" x=\"0\" y=\"360\">disjoint pairs</text><text fill=\"rgb(200,200,200)\" style=\"white-space: pre\" textLength=\"4780\" x=\"300\" y=\"360\">----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------</text><text fill=\"rgb(91,146,121)\" style=\"white-space: pre\" textLength=\"310\" x=\"2930\" y=\"360\">?????????????????????????????????????????????????????????????????????????????????????????????</text><text fill=\"rgb(91,146,121)\" style=\"white-space: pre\" textLength=\"310\" x=\"3240\" y=\"360\">?????????????????????????????????????????????????????????????????????????????????????????????</text><text fill=\"rgb(0,0,0)\" style=\"white-space: pre\" textLength=\"80\" x=\"0\" y=\"400\">example2</text><text fill=\"rgb(0,0,0)\" style=\"white-space: pre\" textLength=\"4840\" x=\"300\" y=\"400\">ATGCCGGACGGATCCGGATCTCGCGGGATCTCGCACCGGATCCGGATCTTTCATAACTATACTCGTAAGGATCATGTTATTGATTTCTTCAAAGGTTACTGTGGGTCTCCGGGCCCCCCCGTCCACCCAGACCGACCGAACTCTCCCGTTTAGGACCCTAGTAAGTCATCATTGGTATATGAATGCGACCCCGAAGAACTGTGGGTCTCCGGGCCCCCCCGTCCACCCAGACCGACCGAACTCTCCCGCAATTCTATAAGAATGCACACTGCATCGATACATAAAACGTCTCGATCGCGCCGGGAAAGGTACGCACGCGGTATATACCGCGTGCGTACCTTTCCCGGCTCCTTCCAGAGGTATGTGGCTGCGTGGTCAAAAGTGCGGCATTCGTATTTGCTCCTCGTGTTTACTCTCACAAACTTGACCTGGAGATCAAGGAGATGCTTCTTGTGGAACTGGACAACGCAGTCCGGATCGCGAA</text><text fill=\"rgb(0,0,0)\" style=\"white-space: pre\" textLength=\"4850\" x=\"300\" y=\"416\">1        10        20        30        40        50        60        70        80        90        100       110       120       130       140       150       160       170       180       190       200       210       220       230       240       250       260       270       280       290       300       310       320       330       340       350       360       370       380       390       400       410       420       430       440       450       460       470       480   </text><text fill=\"rgb(100,100,100)\" style=\"white-space: pre\" textLength=\"120\" x=\"0\" y=\"456\">interspersed</text><text fill=\"rgb(200,200,200)\" style=\"white-space: pre\" textLength=\"4840\" x=\"300\" y=\"456\">----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------</text><text fill=\"rgb(63,106,115)\" style=\"white-space: pre\" textLength=\"510\" x=\"1270\" y=\"456\">?????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????</text><text fill=\"rgb(63,106,115)\" style=\"white-space: pre\" textLength=\"510\" x=\"2270\" y=\"456\">?????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????</text><text fill=\"rgb(100,100,100)\" style=\"white-space: pre\" textLength=\"120\" x=\"0\" y=\"480\">interspersed</text><text fill=\"rgb(200,200,200)\" style=\"white-space: pre\" textLength=\"4840\" x=\"300\" y=\"480\">----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------</text><text fill=\"rgb(63,106,115)\" style=\"white-space: pre\" textLength=\"130\" x=\"380\" y=\"480\">???????????????????????????????????????</text><text fill=\"rgb(63,106,115)\" style=\"white-space: pre\" textLength=\"80\" x=\"400\" y=\"480\">????????????????????????</text><text fill=\"rgb(63,106,115)\" style=\"white-space: pre\" textLength=\"130\" x=\"660\" y=\"480\">???????????????????????????????????????</text><text fill=\"rgb(63,106,115)\" style=\"white-space: pre\" textLength=\"80\" x=\"5010\" y=\"480\">????????????????????????</text><text fill=\"rgb(100,100,100)\" style=\"white-space: pre\" textLength=\"110\" x=\"0\" y=\"504\">palindromic</text><text fill=\"rgb(200,200,200)\" style=\"white-space: pre\" textLength=\"4840\" x=\"300\" y=\"504\">----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------</text><text fill=\"rgb(250,199,72)\" style=\"white-space: pre\" textLength=\"390\" x=\"3140\" y=\"504\">?????????????????????????????????????????????????????????????????????????????????????????????????????????????????????</text><text fill=\"rgb(250,199,72)\" style=\"white-space: pre\" textLength=\"390\" x=\"3530\" y=\"504\">?????????????????????????????????????????????????????????????????????????????????????????????????????????????????????</text><text fill=\"rgb(100,100,100)\" style=\"white-space: pre\" textLength=\"70\" x=\"0\" y=\"528\">high GC</text><text fill=\"rgb(200,200,200)\" style=\"white-space: pre\" textLength=\"4840\" x=\"300\" y=\"528\">----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------</text><text fill=\"rgb(255,66,63)\" style=\"white-space: pre\" textLength=\"400\" x=\"310\" y=\"528\">????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????</text><text fill=\"rgb(255,66,63)\" style=\"white-space: pre\" textLength=\"400\" x=\"1230\" y=\"528\">????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????</text><text fill=\"rgb(255,66,63)\" style=\"white-space: pre\" textLength=\"400\" x=\"2130\" y=\"528\">????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????</text><text fill=\"rgb(100,100,100)\" style=\"white-space: pre\" textLength=\"60\" x=\"0\" y=\"552\">low GC</text><text fill=\"rgb(200,200,200)\" style=\"white-space: pre\" textLength=\"4840\" x=\"300\" y=\"552\">----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------</text><text fill=\"rgb(91,192,255)\" style=\"white-space: pre\" textLength=\"400\" x=\"770\" y=\"552\">????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????</text><text fill=\"rgb(100,100,100)\" style=\"white-space: pre\" textLength=\"100\" x=\"0\" y=\"576\">self pairs</text><text fill=\"rgb(200,200,200)\" style=\"white-space: pre\" textLength=\"4840\" x=\"300\" y=\"576\">----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------</text><text fill=\"rgb(168,58,36)\" style=\"white-space: pre\" textLength=\"310\" x=\"3400\" y=\"576\">?????????????????????????????????????????????????????????????????????????????????????????????</text><text fill=\"rgb(168,58,36)\" style=\"white-space: pre\" textLength=\"310\" x=\"3400\" y=\"576\">?????????????????????????????????????????????????????????????????????????????????????????????</text><text fill=\"rgb(100,100,100)\" style=\"white-space: pre\" textLength=\"140\" x=\"0\" y=\"600\">disjoint pairs</text><text fill=\"rgb(200,200,200)\" style=\"white-space: pre\" textLength=\"4840\" x=\"300\" y=\"600\">----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------</text><text fill=\"rgb(91,146,121)\" style=\"white-space: pre\" textLength=\"310\" x=\"300\" y=\"600\">?????????????????????????????????????????????????????????????????????????????????????????????</text><text fill=\"rgb(91,146,121)\" style=\"white-space: pre\" textLength=\"310\" x=\"2310\" y=\"600\">?????????????????????????????????????????????????????????????????????????????????????????????</text><text fill=\"rgb(100,100,100)\" style=\"white-space: pre\" textLength=\"140\" x=\"0\" y=\"624\">disjoint pairs</text><text fill=\"rgb(200,200,200)\" style=\"white-space: pre\" textLength=\"4840\" x=\"300\" y=\"624\">----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------</text><text fill=\"rgb(91,146,121)\" style=\"white-space: pre\" textLength=\"310\" x=\"1230\" y=\"624\">?????????????????????????????????????????????????????????????????????????????????????????????</text><text fill=\"rgb(91,146,121)\" style=\"white-space: pre\" textLength=\"310\" x=\"2310\" y=\"624\">?????????????????????????????????????????????????????????????????????????????????????????????</text><text fill=\"rgb(100,100,100)\" style=\"white-space: pre\" textLength=\"140\" x=\"0\" y=\"648\">disjoint pairs</text><text fill=\"rgb(200,200,200)\" style=\"white-space: pre\" textLength=\"4840\" x=\"300\" y=\"648\">----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------</text><text fill=\"rgb(91,146,121)\" style=\"white-space: pre\" textLength=\"310\" x=\"1380\" y=\"648\">?????????????????????????????????????????????????????????????????????????????????????????????</text><text fill=\"rgb(91,146,121)\" style=\"white-space: pre\" textLength=\"310\" x=\"2160\" y=\"648\">?????????????????????????????????????????????????????????????????????????????????????????????</text><text fill=\"rgb(100,100,100)\" style=\"white-space: pre\" textLength=\"140\" x=\"0\" y=\"672\">disjoint pairs</text><text fill=\"rgb(200,200,200)\" style=\"white-space: pre\" textLength=\"4840\" x=\"300\" y=\"672\">----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------</text><text fill=\"rgb(91,146,121)\" style=\"white-space: pre\" textLength=\"310\" x=\"1380\" y=\"672\">?????????????????????????????????????????????????????????????????????????????????????????????</text><text fill=\"rgb(91,146,121)\" style=\"white-space: pre\" textLength=\"310\" x=\"2470\" y=\"672\">?????????????????????????????????????????????????????????????????????????????????????????????</text><text fill=\"rgb(100,100,100)\" style=\"white-space: pre\" textLength=\"140\" x=\"0\" y=\"696\">disjoint pairs</text><text fill=\"rgb(200,200,200)\" style=\"white-space: pre\" textLength=\"4840\" x=\"300\" y=\"696\">----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------</text><text fill=\"rgb(91,146,121)\" style=\"white-space: pre\" textLength=\"310\" x=\"1540\" y=\"696\">?????????????????????????????????????????????????????????????????????????????????????????????</text><text fill=\"rgb(91,146,121)\" style=\"white-space: pre\" textLength=\"310\" x=\"2620\" y=\"696\">?????????????????????????????????????????????????????????????????????????????????????????????</text><text fill=\"rgb(100,100,100)\" style=\"white-space: pre\" textLength=\"140\" x=\"0\" y=\"720\">disjoint pairs</text><text fill=\"rgb(200,200,200)\" style=\"white-space: pre\" textLength=\"4840\" x=\"300\" y=\"720\">----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------</text><text fill=\"rgb(91,146,121)\" style=\"white-space: pre\" textLength=\"310\" x=\"3240\" y=\"720\">?????????????????????????????????????????????????????????????????????????????????????????????</text><text fill=\"rgb(91,146,121)\" style=\"white-space: pre\" textLength=\"310\" x=\"3550\" y=\"720\">?????????????????????????????????????????????????????????????????????????????????????????????</text></g></svg>",
   "Other":{
      "emails":"your@email.com",
      "Job Id":"b2816ba3-3390-437b-a37c-03fc1238db94"
   },
   "Finished":"b2816ba3-3390-437b-a37c-03fc1238db94"
}
```


## RESTful Web Service
- Secure network interface
- Can be incorporated into various computational pipelines
- Can be used on the public facing internet or internally on a private intranet
- Provides JSON as input and output
- Could interact with customer databases

### Example Python Script for RESTful API Call to Sequence Complexity:
``` python3
import requests
import time

url = "https://homologypath.com/router/"
token = "YOUR_API_TOKEN_HERE"
data = {
    "api": "sequence_complexity",
    "required_parameters": {
        "sequences": [
            {
                "sequence_name": "example1",
                "sequence": "ATGGGGACCGGATCCGGATCGGGTTTCATAACTATACTCGTAAGGATCATGTTATTGATTTCTTCAAAGGTTACTGTGGGTCTCCGGGCCCCCCCGTCCACCCAGACCGACCGAACTCTCCCGTTTAGGACCCTAGTAAGTCATCATTGGTATATGAATGCGACCCCGAAGAACTGTGGGTCTCCGGGCCCCCCCGTCCACCCAGACCGACCGAACTCTCCCGCAATTCTATAAGAATGCACACTGCATCGATACATAAAACGTCTCGATCGCGCCGGGAAAGGTACGCACGCGGTATATACCGCGTGCGTACCTTTCCCGGCTCCTTCCAGAGGTATGTGGCTGCGTGGTCAAAAGTGCGGCATTCGTATTTGCTCCTCGTGTTTACTCTCACAAACTTGACCTGGAGATCAAGGAGATGCTTCTTGTGGAACTGGACAACGCAATCTCGCGCCCGGATCCGGATCGGAAAGCGAAA",
            },
            {
                "sequence_name": "example2",
                "sequence": "ATGCCGGACGGATCCGGATCTCGCGGGATCTCGCACCGGATCCGGATCTTTCATAACTATACTCGTAAGGATCATGTTATTGATTTCTTCAAAGGTTACTGTGGGTCTCCGGGCCCCCCCGTCCACCCAGACCGACCGAACTCTCCCGTTTAGGACCCTAGTAAGTCATCATTGGTATATGAATGCGACCCCGAAGAACTGTGGGTCTCCGGGCCCCCCCGTCCACCCAGACCGACCGAACTCTCCCGCAATTCTATAAGAATGCACACTGCATCGATACATAAAACGTCTCGATCGCGCCGGGAAAGGTACGCACGCGGTATATACCGCGTGCGTACCTTTCCCGGCTCCTTCCAGAGGTATGTGGCTGCGTGGTCAAAAGTGCGGCATTCGTATTTGCTCCTCGTGTTTACTCTCACAAACTTGACCTGGAGATCAAGGAGATGCTTCTTGTGGAACTGGACAACGCAGTCCGGATCGCGAA",
            },
        ],
        "sequence_complexity_parameters": {
            "minimum_length": "25",
            "target_length": "30",
            "maximum_length": "40",
            "minimum_overlap": "15",
            "maximum_overlap": "25",
            "minimum_zscore_cutoff": "-4.0",
            "temp": "60",
        },
    },
}

# Post a request with job info to the API.
print("Posting request (worker starting may take some time)...")
response = requests.post(
    url,
    headers={"Authorization": f"Token {token}"},
    json=data,
)
data = response.json()

# At this point "job_id" field means the worker is booted and computing.
print("Response:", data)

# Depending on how big the request was, you should periodically poll the API with "job_id".
# This example just sleeps for 10 seconds, which is enough time to finish the calculations.
time.sleep(10)

# Poll the API to get the job results.
print(f"Checking {data} for results...")
data["api"] = "sequence_complexity"
response = requests.post(
    url,
    headers={"Authorization": f"Token {token}"},
    json=data,
)
response = response.json()

# A polling response with successful execution will have a "Finished" field with the "job_id".
if "Finished" in response:
    print(f"Finished {data} with:\n", response)
else:
    print(f"Not finished, make another request with better parameters!")

```
