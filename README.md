# Tisk 1.x Distribution

This code is to run a TISK model on Python 3.x.

# Files
There are three files in TISK 1.x.

Basic_TISK_Class.py<br>
Phoneme_Data.txt<br>
Pronunciation.txt<br>

# Pre-installed

This code require 'numpy', 'matplotlib' modules.
```
pip install numpy<br>
pip install matplotlab<br>
```

# TISK 1.0 code example
## Running TISK
### Preliminary steps
The following commands prepare TISK for simulations. Lines preceded by "#" are comments and can be skipped.
```
# load the TISK functions
import Basic_TISK_Class as tisk

# load the phoneme and pronunciation [word] lists and 
#prepare appropriate connections
phoneme_List, pronunciation_List = tisk.List_Generate()

# initialize the model with the the phoneme_List, 
# pronunciation_List, number of time slots, and threshold
tisk_Model = tisk.TISK_Model(phoneme_List, pronunciation_List,
                             time_Slots = 10,
                             nPhone_Threshold = 0.91)
```
### Initialize and / or modify parameters

Before running simulations, you must initialize the parameters. To use the default parameters of TISK 1.0 (Hannagan et al., 2013), just enter: 
```
# initialize the model with default or current parameters
tisk_Model.Weight_Initialize()
```

The model does not automatically initialize because this is the step where all connections are made, etc., and initialization can take a long time for a large model with thousands of words. To control specific categories of parameters or specific parameters, use the following examples: 
```
# change selected TISK parameters
tisk_Model.Decay_Parameter_Assign(
                    decay_Phoneme = 0.001,
                    decay_Diphone = 0.001,
                    decay_SPhone = 0.001,
                    decay_Word = 0.01)
tisk_Model.Weight_Parameter_Assign(
                    input_to_Phoneme_Weight = 1.0,
                    phoneme_to_Phone_Weight = 0.1,
                    diphone_to_Word_Weight = 0.05,
                    sPhone_to_Word_Weight = 0.01,
                    word_to_Word_Weight = -0.005)
tisk_Model.Feedback_Parameter_Assign(
                    word_to_Diphone_Activation = 0,
                    word_to_SPhone_Activation = 0,
                    word_to_Diphone_Inhibition = 0,
                    word_to_SPhone_Inhibition = 0)
tisk_Model.Weight_Initialize()
```

To modify a subset of parameters, just specify the subset, and the others will retain their current values. For example: 

```
tisk_Model.Decay_Parameter_Assign(
                    decay_Phoneme = 0.002,
                    decay_Diphone = 0.0005)
```

To list the current parameters, enter the following command:

```
tisk_Model.Parameter_Display()
```

## Simulate processing of a phoneme string and graph results for phonemes and words
Here is an example of a basic command that calls a simulation of the word 'pat' (technically, it is more correct to say "a simulation of the pronunciation 'pat'", since the user can specify pronunciations that are not in the lexicon [i.e., the pronunciation_List]):

```
# trigger a simulation without producing output;
# this prepares a model for inspection
tisk_Model.Display_Graph(pronunciation='pat')
```

On its own, this command doesn't do anything apparent to the user (though the simulation is in fact conducted). To create a graph that is displayed within an IDE, add arguments to display specific phonemes:

```
# trigger a simulation and create a phoneme input graph
tisk_Model.Display_Graph(
           pronunciation='pat',
           display_Phoneme_List = [('p', 0), ('a', 1), ('t', 2)])
```

This code means "input the pronunciation /pat/, and export a graph with phoneme activations for /p/, /a/, and /t/ in the first, second, and third positions, respectively".

We can extend this to create activation graphs of diphones, single phones, and words. The following example creates one of each:

```
# trigger a simulation and make 3 graphs
tisk_Model.Display_Graph(pronunciation='pat',
                         display_Diphone_List = ['pa', 'pt', 'ap'],
                         display_Single_Phone_List = ['p', 'a', 't'],
                         display_Word_List = ['pat', 'tap'])
```

To export the graphs in a standard graphics format (PNG), simply add one more argument to the command:
 
```
# trigger a simulation, make 3 graphs, save them as PNG files
tisk_Model.Display_Graph(pronunciation='pat',
                         display_Diphone_List = ['pa', 'pt', 'ap'],
                         display_Single_Phone_List = ['p', 'a', 't'],
                         display_Word_List = ['pat', 'tap'],
                         file_Save = True)
```

## Extract simulation data to a numpy matrix

The basic method for extracting data as a numpy matrix is as follows:
```
# trigger a simulation and ready data structures
# without creating any output
tisk_Model.Extract_Data(pronunciation='pat')
```

The method is similar to the graphing functions. The code above is the core command that readies appropriate structures for extraction, but it does not by itself generate any result for the user. To extract data, arguments specifying the details desired are required. For example, to get data corresponding to the word plot above (showing the activations of /pat/ and /tap/ given the input /pat/), the following command would put the data in a numpy matrix called 'result':

```
# trigger a simulation and assign data structures to 'result'
result = tisk_Model.Extract_Data(pronunciation='pat',
                                 extract_Word_List = ['pat', 'tap'])
```

When this command is executed, the 'result' variable becomes a list with length 1, consisting of a single numpy matrix with shape (2, 100). The first and second rows are the activation patterns of the word units for /pat/ and /tap/, respectively.

```
# trigger a simulation and assign data structures to 'result'
result = tisk_Model.Extract_Data(pronunciation='pat',
            extract_Phoneme_List = [('p', 0), ('a', 1), ('t', 2)],
            extract_Single_Phone_List = ['p', 'a', 't'])
```

Here, result becomes a list with length 2. The first item is a numpy matrix with the input unit activations for the 3 specified phonemes across the 100 steps of the simulation. The second is a numpy matrix with the activations of the specified single phonemes in the n-phone layer over the 100 steps of the simulation.

## Export simulation data to text files

To export results to text files, we add a parameter:

```
# trigger a simulation and save data structures to text file
result = tisk_Model.Extract_Data(pronunciation='pat',
         extract_Word_List = ['pat', 'tap'], file_Save=True)
```

This creates a text file called "p_a_t.Word.txt". The file has 102 columns and 3 lines. The first line is a header, labeling the columns; the subsequent lines contain the data. The first column is the input string ("p a t"), and the second is the specified word to track (line 2 is "pat" and line 3 is "tap"). Columns 3-102 are activations for the corresponding word in cycles 0-99.

## Batch simulation of multiple words
```
# get mean RT and accuracy for the specified set of words
rt_and_ACC = tisk_Model.Run_List(pronunciation_List = ['baks','bar','bark','bat^l','bi'])
```
Given this command, the model will simulate the 5 words and check the reaction time (RT) and accuracy for each. The variable 'acc_and_RT' will be a list of 6 items, with mean RT and accuracy for the specified words computed using three different methods (abs = based on an absolute threshold [target must exceed threshold], rel = relative threshold [target must exceed next most active item by threshold], tim = time-based threshold [target must exceed next most active item by threshold for at least a specified number of cycles]):

```
rt_and_ACC[0]: Mean of RT~abs~
rt_and_ACC[1]: Mean of ACC~abs~
rt_and_ACC[2]: Mean of RT~rel~
rt_and_ACC[3]: Mean of ACC~rel~
rt_and_ACC[4]: Mean of RT~tim~
rt_and_ACC[5]: Mean of ACC~tim~
```

More commonly, one might want to evaluate mean accuracy and RT for every word in the current lexicon with the current parameter settings. The following command would do this, where we specify the pronunciation_List to be the full pronunciation_List: 

```
# get mean RT and accuracy for all words in pronunciation_List
rt_and_ACC = tisk_Model.Run_List(
          pronunciation_List = pronunciation_List)
```

The parameters used for the different accuracy methods can also be modified. The default criteria are: abs = 0.75, rel = 0.05, tim = 10 (time steps). These criteria refer to absolute activation values (to "win", a target's activation must exceed 0.75), relative activation values (to win, the target's activation must exceed all other words' activations by at least 0.05, and time steps (to win, the target must have the highest activation, and its activation must exceed that of all other words' activations for at least 10 time steps). Here is an example where the criteria for each accuracy method are specified:

```
# get mean RT and accuracy for specified word list with 
# specified accuracy criteria for abs, rel, and tim, respectively
rt_and_ACC = tisk_Model.Run_List(
        pronunciation_List = ['baks','bar','bark','bat^l','bi'],
        absolute_Acc_Criteria=0.6,
        relative_Acc_Criteria=0.01,
        time_Acc_Criteria=5)
```

Often, we may want to obtain the RT values for each word in a list, rather than the mean values. We can do this using the reaction_Time flag with the Run_List procedure. Currently, this requires you to specify a file to write the data to (which could be read back in using standard Python techniques): 

```
tisk_Model.Run_List(pronunciation_List = ['baks','bar','bark','bat^l','bi'], 
                    output_File_Name = "Test", 
                    reaction_Time=True)
```

This will create an output file named "Test_Reaction_Time.txt". Its contents would be:

| Target | Absolute | Relative | Time_Dependent |
|--------|----------|----------|----------------|
| baks   | 58       | 40       | 46             |
| bar    | 84       | 28       | 33             |
| bark   | 74       | 52       | 56             |
| bat^l  | 60       | 39       | 46             |
| bi     | nan      | 23       | 13             |

Accuracy is indicated by the value for each word for each accuracy criterion. Items that were correctly recognized according to the criterion will have integer values (cycle at which the criterion was met). Items that were not will have values of "nan" (not a number, a standard designation for a missing value). In the current example, we can see that /bi/ ("bee") did not meet the absolute criterion. 

If you wanted to find obtain the RTs for every word in your lexicon, you would replace the word list with pronunciation_List:

```
tisk_Model.Run_List(pronunciation_List = pronunciation_List, 
                    output_File_Name = "all", 
                    reaction_Time=True)
```

## Extract data for multiple words in text files

To export results with multiple words, we can use the 'Run_List' function again, as follows: 

```
# get mean RT and accuracy for specified word list
# with specified accuracy criteria but ALSO 
# save activation histories in 'raw' and 'category' formats
rt_and_ACC = tisk_Model.Run_List(
        pronunciation_List = ['baks','bar','bark','bat^l','bi'],
        output_File_Name = 'Result',
        raw_Data = True,
        categorize=True)
```

When we run this code, we get text files with what we call 'raw' and 'category' outputs. Raw files (e.g., for this example, Result_Word_Activation_Data.txt) contain the activations for every word in the lexicon at every time step for each target specified. The file format is very simple. There is a 1-line header with column labels. The first column is 'target', the second is 'word', and the following columns are cycles 0-C, where C is the final cycle (which will have the value [(time_Slots x IStep_Length) -1].

You can also make graphs that correspond to the category data. For example, let's plot the category data for /baks/. 

```
# trigger a simulation and make a graph
tisk_Model.Average_Activation_by_Category_Graph(
          pronunciation_List=['baks'])
```

We can also get average data and average plots for a set of specified words. For example, suppose for some reason we were interested in the average category plot for the words /pat/, /tap/, and /art/ ("pot", "top", and "art").

```
# trigger a simulation and make a graph
tisk_Model.Average_Activation_by_Category_Graph(
          pronunciation_List=['pat', 'tap', 'art'])
```

To save this graph as a PNG file, add the file_Save=True argument: 

```
# trigger a simulation, make a graph, save them as PNG files
tisk_Model.Average_Activation_by_Category_Graph(
          pronunciation_List=['pat','tap', 'art'],
          file_Save=True)
```

By default, the graph associated with this command will be saved as "Average_Activation_by_Category_Graph.png". To specify a different filename (important if you wish, for example, to loop through many example sets in a Python script), you can do so as follows: 

```
# trigger a simulation, make a graph, save them as PNG files
tisk_Model.Average_Activation_by_Category_Graph(
          pronunciation_List=['pat','tap', 'art'],
          output_File_Name='Result',
          file_Save=True)
```

In this case, the exported graph file name will become 'Result_Average_Activation_by_Category_Graph.png'. By setting the output_File_Name parameter, you can control the prefix of exported file name.

To save the corresponding data file in text format, use the Run_List function: 

```
tisk_Model.Run_List(
          pronunciation_List=['pat','tap', 'art'],
          output_File_Name='Result',
          categorize=True)
```

## Getting comprehensive data for every word in the lexicon

If we combine our last few examples, we can save activations for simulations of every word by replacing our pronunication_List argument above with pronunciation_List (that is, all words included in pronunciation_List):

```
rt_and_ACC = tisk_Model.Run_List(
          pronunciation_List = pronunciation_List,
          output_File_Name = 'all_words',
          raw_Data = True,
          categorize=True)
```

By extension, we can also generate a mean category plot over every word in the lexicon: 

```
# make a graph for all words in pronunciation_List
tisk_Model.Average_Activation_by_Category_Graph(
pronunciation_List = pronunciation_List)
```

## Batch size control

Depending on the size of your lexicon and the memory available on your computer, you may see the 'Memory Error' message when you run batch mode. Batch-mode simulation is not possible if the memory of the machine is too small to handle the size of the batch. To resolve this, you can use the batch_Size parameter to reduce the size of the batch. This parameter determines how many word simulations are conducted in parallel. It only controls the batch size, and does not affect any result. You will get the same result with any batch size your computer's memory can handle. The default value is 100. To see whether your computer memory can handle it, you can test larger values. 

```
rt_and_ACC = tisk_Model.Run_List(
          pronunciation_List = pronunciation_List,
          batch_Size = 10)
```

## Reaction time and accuracy for specific words

To check specific kinds of RT for specific words, use commands like these: 

```
result = tisk_Model.Run('pat')
abs_RT = tisk_Model.RT_Absolute_Threshold(
                  pronunciation = 'pat',
                  word_Activation_Array = result[3],
                  criterion = 0.75)
rel_RT = tisk_Model.RT_Relative_Threshold(
                  pronunciation = 'pat',
                  word_Activation_Array = result[3],
                  criterion = 0.05)
tim_RT = tisk_Model.RT_Time_Dependent(
                  pronunciation = 'pat',
                  word_Activation_Array = result[3],
                  criterion = 10)
```

If TISK successfully recognized the inserted word, the reaction time will be returned. If the model failed to recognize the word, the returned value is 'numpy.nan'. Of course, we can change the criterion by modifying the parameter 'criterion'.

Alternatively, we could get all accuracy and RT values for a specific word by using a command we introduced earlier:

```
rt_and_ACC = tisk_Model.Run_List(pronunciation_List = ['pat'])
```

## More complex simulations

Since TISK is implemented as a Python class, the user can do arbitrarily complex simulations by writing Python scripts. Doing this may require the user to acquire expertise in Python that is beyond the scope of this short introductory guide. However, to illustrate how one might do this, we include one full, realistic example here. In this example, we will compare competitor effects as a function of word length, by comparing competitor effects for words that are 3 phonemes long vs. words that are 5 phonemes long. All explanations are embedded as comments (preceded by "#") in the code below:

```
# first, select all words that have length 3 in the lexicon
length3_Pronunciation_List = [x for x in pronunciation_List if len(x) == 3]

# now do the same for words with length 5
length5_Pronunciation_List = [x for x in pronunciation_List if len(x) == 5]

# make a graph of average competitor effects for 3-phoneme words
tisk_Model.Average_Activation_by_Category_Graph(
pronunciation_List = length3_Pronunciation_List)

# make a graph and also save to a PNG file
tisk_Model.Average_Activation_by_Category_Graph(
pronunciation_List= length3_Pronunciation_List,
file_Save=True,
output_File_Name='length_3_category_results.png')

# make a graph and also save to a PNG file
tisk_Model.Average_Activation_by_Category_Graph(
pronunciation_List= length5_Pronunciation_List,
file_Save=True,
output_File_Name= 'length_5_category_results.png')

# save the length 3 data
tisk_Model.Run_List( pronunciation_List = length3_Pronunciation_List, 
output_File_Name='length3data', 
raw_Data = True, categorize = True)
```

Note that when you save data to text files, if you leave out the " categorize = True " argument, the file with word results will include the over time results for every target in pronunciation list. The first column will list the target, the second column the time step, and then there will be 1 column for every word in the lexicon (i.e., with the activation of that word given the current target at the specified time step).

Of course, this example just scratches the surface of what is possible since TISK is embedded within a complete scripting language. Using standard Python syntax, we can easily filter words by specifying arbitrarily complex conditions. Here are some examples:

```
# Select words with length greater than or equal to 3 with 
# phoneme /a/ in position 2
filtered_Pronunciation_List = [x for x in pronunciation_List if len(x) >= 3 and x[2] == 'a']

# make a graph
tisk_Model.Average_Activation_by_Category_Graph(
pronunciation_List = filtered_Pronunciation_List)

# select words with length greater than or equal to 2 with 
# phoneme /a/ in position 2 and phoneme /k/ in position 3
filtered_Pronunciation_List = [x for x in pronunciation_List if len(x) >= 4 and x[2] == 'a' and x[3]=='k']
```

# Reporting Issues

If you suspect that a bug has caused a malfunction while using the program, please report it using the issues feature of this repository.

https://github.com/CODEJIN/Tisk/issues

However, the current status of this project is occasional management. Therefore, it is difficult to respond promptly to reported issues. Thank you for your understanding.
