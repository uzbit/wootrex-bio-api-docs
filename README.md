# wootrex bio API Documentation

## Oligo Design

The Oligo Design bioinformatics software package and API provide a simple and intuitive method to create single stranded oligonucleotide sequences that, when pooled together, assemble into the requested sequence. It also provides an ability to take a library of sequences with homologous and variable regions and create a design across the entire library that minimizes the number of homologous region oligos, while also preserving the required variable region oligos. Primers for amplification are additionally produced and recycled where necessary. Both the oligo design and the primer design are presented with their respective plate maps for ease of automation in the wet lab. The output is easily consumed by all major oligo manufacturers such as IDT, Twist, etc. 

### Parameters
The API accepts a JSON object containing the following:

#### API
A required string specifying which API is being requested. Valid strings are: "oligo_design", "sequence_complexity", "sequence_design"

#### Required Parameters
A required JSON object containing all three of the following entries:

###### Sequence Records
A list of JSON objects of sequence records that include the following:

1. **Sequence Name**: String, a unique name for each sequence.
2. **Sequence**: String, ambiguous/unambiguous DNA sequence.

###### Oligo Design Parameters
A JSON object containing the following:

1. **Minimum Length**: Integer, default 30
The minimum size in base pairs allowed for any oligo in the design. 
2. **Target Length**: Integer, default 50 
The target size in base pairs for an oligo to start with during design. 
3. **Maximum Length**: Integer, default 60 
The maximum size in base pairs allowed for any oligo in the design. 
4. **Minimum Overlap**: Integer, default 15
The minimum number of basepairs allowed for any given overlap between 3’ and 5’ paired oligos.
5. **Maximum Overlap**: Integer, default 25
The maximum number of basepairs allowed for any given overlap between 3’ and 5’ paired oligos.

###### Primer Design Parameters 
A JSON object containing the following:

1. **Target Primer Length**: Integer, default 20
The minimum size in base pairs allowed for any oligo in the design. 
2. **TM Optimize Primers**: Boolean, default True
Wether or not to melting temperature (TM) optimize the amplification primers. 
3. **Target Primer Temperature**: Integer, default 60
The temperature (in degrees Celcius) at wich PCA is performed. This is only used if **TM Optimize Primers** is **True**


#### Optional Parameters
An optional JSON object containing any of the following:

1. **Recycle Oligos**: Boolean, default True.
Perform oligo recycling across the entire set of sequences provided in 
Sequence Records.
2. **Source Plate Size**: Integer, default 96.
Number of wells of the oligo plate to order from the oligo manufacturer. Currently must be either 96 or 384. 
3. **Destination Plate Size**: Integer, default 96.
Number of wells of the pooling plate. Currently must be either 96 or 384. 
4. **Row Major**: Boolean, default True
Provide the list of oligos in row first (A1, A2, A3, etc) order if True, otherwise provide this list in column first order (A1, B1, C1, etc).

### Output
An object that contains the oligo and primer designs and plate maps in one of XLSX, JSON or some other requested format. See below for an example of the JSON output.

### Error Handling
If the design parameters are invalid or result in an invalid design (low success of assembly, or doesn’t match parameters) verbose errors are returned for resolution.

### Use Cases
#### Homology Path
- Oligo Design is fully implemented in the Ninth Bio Homology Path software suite.
#### RESTful Web Service
- Secure network interface
- Could be on the internet or private network
- Could take JSON as input and output
- Could interact with customer databases
#### Command Line
- Same advantages as above, but additionally can run on a network isolated computer
Library


### Technical Examples

```
curl -0 -X POST "https://homologypath.com/router/" \
-H "Authorization: Token 2fb15ad154d14b466600d43b0d586b18e247426f" \
-H "Content-Type: application/json" \
-d \
'
{
    "api":"oligo_design",
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
       "emails":"guzbit@gmail.com",
       "row_major":"true",
       "recycle_oligos":"true",
       "partition_identity":"0.95"
    }
 }'

```



## Sequence Design
The Sequence Complexity bioinformatics software package and API provide a simple and intuitive method to analyze DNA sequences for regions that may make a given sequence more difficult to synthesize. This package looks for repeat regions of various sorts, high and low GC content regions, and most importantly performs a simulated analysis of a given oligo design based on the Gibbs Free Energy (ΔG) of the pool of oligos. An example output from the Sequence Complexity package can be seen below. The sequence shown has two large interspersed repeats (dark green), high (red) and low (blue) GC regions, and a large palindromic/hairpin region (yellow). The lighter green segments (disjoint oligo pairs) at the bottom represent oligos in a design that have an abnormally high affinity (more negative ΔG), yet were not intended to anneal. This would likely result in truncated products during assembly. There is also a dark red segment (self oligo pairs) in the region of the hairpin. This oligo is likely to fold on itself and hence not be available for the assembly, resulting again in truncated product. Using this information, and the Oligo Design tool, one should be able to create a design that minimizes these synthesis issues. 

