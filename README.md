# a3mtools

Tools for working with a3m files. Designed to help generate input for structure prediction tools like alphafold. 

**Main features:**
- import a3m files from MMseqs2 search results into python objects
- Easily slice MSAs while preserving insertions
- Combine multiple MSAs into a single a3m file for complex prediction
- Save manipulated MSAs to new a3m files

<!-- Provides a method to easily slice a3m MSAs while preserving insertions. <br>Also allows you to concatenate MSAs into a single a3m file for complex prediction. -->

## Installation
```bash
pip install a3mtools
```

or if you want an editable version:
```bash
git clone https://github.com/jacksonh1/a3mtools.git
cd a3mtools
pip install -e .
```

## What these tools do:

The current implementation of the tools are designed to handle MSAs retrieved using the colabfold MMseqs2 search tool. There are some strange quirks about those files specifically that I tried to account for. Mainly query sequence names are denoted by 3 digit integers starting at 101. So if an MSA only has 1 query sequence it will be named 101. If there are 2 query sequences they will be named 101 and 102 etc. This comes into play when combining MSAs for predicting protein complexes. Additionally, when combining 2 MSAs, the query sequences are combined into a single sequence. This is required to predict the complex structure. The query sequences are also added back to the MSA as individual sequences. **MSAs are combined in unpaired format**.<br>
<br>
Here's an example (with no homologous sequences):
<br>
MSA1 a3m file:
```
#9	1
>101
ABCDEFGHI
```
MSA2 a3m file:
```
#17	1
>101
JKLMNOPQRSTUVWXYZ
```
MSA1 + MSA2 a3m file:
```
#9,17   1,1
>101    102
ABCDEFGHIJKLMNOPQRSTUVWXYZ
>101
ABCDEFGHI-----------------
>102
---------JKLMNOPQRSTUVWXYZ
```
The MSA1 + MSA2 a3m file could be used as input to many structure prediction tools to predict the structure of the complex formed by the 2 query sequences. <br>

I don't know if maintaining this specific 101, 102, 103 ... naming scheme is strictly necessary, but we'll stick with it for now. <br>

The tools also allow for slicing the MSA and saving the MSA to a file. <br>
When slicing or combining MSAs, insertions are maintained (see output of examples). <br>



## Usage
There will eventually be a more complete guide. But for now, you can install the package and run the following code to see some of the basic functionality. <br>


```python
import a3mtools
import a3mtools.examples as examples
```

### import an a3m file
```python
msa = a3mtools.MSAa3m.from_a3m_file(examples.example_a3m_file1)
print(msa)
```

```
#9	1
>101
ABCDEFGHI
>ortho1
xxABCDxxEFGZZ
>ortho2
A--DxxE-GHIxxxx
>ortho3
----xxEFGH-
```
The input to the `a3mtools.MSAa3m.from_a3m_file` function is the path to any a3m file.

### slicing the alignment
```python
print(msa[2:5])
```
```
#3	1
>101
CDE
>ortho1
CDxxE
>ortho2
-DxxE
>ortho3
--xxE
```

### concatenating alignments
```python
msa2 = a3mtools.MSAa3m.from_a3m_file(examples.example_a3m_file2)
print(msa2)
```
```
#17	1
>101
JKLMNOPQRSTUVW
>ortho1
JKLMNOPQRSTUVWxxxxxx
>ortho2
------PQRSTUVW
```
<br>

```python
print(msa + msa2)
```
```
#9,17	1,1
>101	102
ABCDEFGHIJKLMNOPQRSTUVW
>101
ABCDEFGHI--------------
>ortho1
xxABCDxxEFGZZ--------------
>ortho2
A--DxxE-GHIxxxx--------------
>ortho3
----xxEFGH---------------
>102
---------JKLMNOPQRSTUVW
>ortho1
---------JKLMNOPQRSTUVWxxxxxx
>ortho2
---------------PQRSTUVW
```
<br>

```python
print(msa + msa2[2:5] + msa[5:])
```
```
#9,3,4	1,1,1
>101	102	103
ABCDEFGHILMNFGHI
>101
ABCDEFGHI-------
>ortho1
xxABCDxxEFGZZ-------
>ortho2
A--DxxE-GHIxxxx-------
>ortho3
----xxEFGH--------
>102
---------LMN----
>ortho1
---------LMN----
>103
------------FGHI
>ortho1
------------FGZZ
>ortho2
-------------GHI
>ortho3
------------FGH-
```

### saving the alignment to a file
```python
complex_msa = msa + msa2
complex_msa.save("example_complex.a3m")
```

## important notes
- slice numbering is relative to the **query sequence**
- query sequence is always the first sequence in the MSA, and is named 101 or some combination of concatenated querys (e.g. 101\t102)
- slicing with a step size other than 1 is not supported yet and probably won't be
- **MSAs are combined in unpaired format**
  - combining MSAs in paired format is not supported yet


## future features:
---
- [ ] documentation on readthedocs
  - [ ] examples and code
- [ ] convert between fasta and a3m and back
- [ ] allow for more generic naming of query sequence
- [ ] add better test functions
- [ ] an option for combining MSAs in paired format

