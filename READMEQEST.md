# QEST experiments

This file contains instructions of how to reproduce the experiments 
included in a paper (or at least its extended version) submitted to QEST.
There are three case studies in the paper, hence the instructions are
split into three parts.

A few things to note:
- All the commands below should be run from the top-level directory, the
same directory in which this README file is located.
- By default, the tool prints quite a lot of output when running 
simulations. To suppress this, append ```--log 1``` to the command
- I prepared and tested the VM image on a 2016 MBP running macOS Big
Sur (Version 11.1), with 8GB of RAM and 2.9 GHz Dual-Core Inter Core 
i5 processor. I also assigned 4GB of RAM to the VM in VirtualBox.

## Trust Game Experiments
Our trust game case study involves three hypotheses, first of which 
(H1) is somewhat trivial and can be confirmed along the second one.
Therefore, two experiments are needed to verify the hypotheses:

1. The first experiment investigates the relationship between initial 
trusts of agents and average transfers. Note that the value of trust
is determined by the value of belief (represented as parameters to a 
Dirichlet distribution). To run this experiment, execute the following
command:
```
webppl examples/trustgame/src/simulations.wppl --require . --require examples/trustgame -- --experiment 1 --log 1
```
(I included ```--log 1``` as otherwise A LOT of output is produced)
This might take 10 minutes or so as it simulates the execution twenty 
times
for each belief configuration and there are five different belief 
configurations. In the unlikely scenario that this takes too long to
run, passing ```--reps 10``` (or substitute 10 with an even smaller 
integer) will change the number of simulations of each configuration 
to 10.

The tool prints traces for each simulation, followed by average
investments and returns for each configurations. The hope is that
the averages will match Fig. 4 from the paper.

Disclaimer: I'll be honest, I have discovered a small bug in the 
implementation of Trust Game (state rewards were being assigned incorrectly)
after submitting the paper. This doesn't affect the simulations 
massively, but might affect them enough that the results of this 
experiment will be somewhat different.

2. The second experiment is about conman behaviour. To run it, execute
```
webppl examples/trustgame/src/simulations.wppl --require . --require examples/trustgame -- --experiment 3 --log 1
```
The tool will print traces from twenty (but ```--reps <int>``` can be 
passed to change that number) simulations along with the average 
income of Alice and Bob ('all 0' traces are ignored for the purposes
of computing averages).
This again takes 5-10 minutes to run (and prints plenty of cache info)
The key observation to make is that Bob's strategy is indeed quite
sophisticated - he cooperates initially, but he always keeps all
the money in the last round. He sometimes (hopefully that will be the
case but nothing can be guaranteed due to randomisation) manages to
deceive Alice, when she falls for his tricks and invests her full 
endowment on last round. Other times, Alice realises what he's up to
and she invests nothing on the last round. Overall, we expect Alice to
end up with higher average income, which partly reflects her priviledged
position in the game (she 'calls the shots') and partly the fact that
she doesn't let herself be manipulated by Bob, most of the time.

## Tipping Experiments

The second set of experiments is quite different in nature as it 
involves learning from data. Therefore, the process will be somewhat
more complex, but we provide scripts to automate it.
The experiments can be divided into two parts, first of which uses 
synthetic data, while the second uses data generated by simulations.

1. The first experiment involves inference from three batches of 
synthetic data, each characterised by low, medium or high tips. The data
files are in ```examples/tipping/data``` and to run inference, execute
```
bash examples/tipping/inferFromSyntheticData syntheticInfer.txt
```
This might take over 10 minutes (but hopefully less than 20) and 
it saves the results of inference to a file 
```syntheticInfer.txt``` in ```examples/tipping/results```. An 
alternative file name may be provided if needed. 
The results will be quite concise: for each inference (there will
be three of those), first comes prior, which is always the same 
(uniform), followed by posterior. The posteriors should roughly 
correspond to Figures 7 and 8 from the paper.

Additionally/optionally, we may perform an experiment briefly
mentioned in the paper where we fix Abi's goal coefficients and
her GASP score and infer only the tipping norm by executing
```
bash examples/tipping/inferFromSyntheticDataFixGoalCoeffsAndGasp syntheticInferFixed.txt
```
The results get saved in ```syntheticInferFixed.txt```. This experiment
should run significantly faster (as there's less inference) and the 
posterior should be characterised by less uncertainty.

Finally, to show that tipping norm cannot be accurately inferred from 
the data alone, without taking into account agent's characteristics, we
focus on ```mediumTips.csv```, the data file characterised by medium tips,
and we show that for various configurations of Abi's goal coefficients
and GASP score, inferred posterior on tipping norm will differ.
To perform this experiment, execute:
```
bash examples/tipping/inferTippingNormFromMediumTips syntheticInferTippingNorm.txt
```
(It may take up to 10 minutes)
This script randomises Abi's goal coefficients and her GASP score ten 
times and infers posterior on tipping norm. A more accurate prior is used 
since only one type of data is considered. Hopefully, the posteriors
will be different. 

2. The second type of experiments involves learning from data generated 
by the tool. We start by generating data:
```
bash examples/tipping/generateDataFiles
```
This should create a subdirectory ```generatedData``` within the 
```data``` directory and inside it, three subdirectories ```5```,
```10``` and ```15```, corresponding to the number of rounds of
tipping game simulated. 

To learn from the generated files, execute the following three commands:
```
bash examples/tipping/inferFromGeneratedData inferFromGenerated5rounds.txt 5 10
bash examples/tipping/inferFromGeneratedData inferFromGenerated10rounds.txt 10 10
bash examples/tipping/inferFromGeneratedData inferFromGenerated15rounds.txt 15 10
```

This will likely take a while, especially the last command (1-2 hours).
Each command processes ten files and prints its progress to the user which
should give one an idea of how long exactly it should take.
It will save the results of inference to the above files and it is important
to use these exact filenames as they are hardcoded in the evaluation script
(see below, it's not an ideal solution but it works...). 

To evaluate the posteriors, we have included a python script which reads
the text files (both data files and results file) to obtain real values
and predictions and computes PMSE (generalisation of MSE to predictions)
for each prediction (separately for goal coefficients, tipping norm and 
gasp score). PMSEs are then averaged, as means and medians and printed
to the user. 
Execute the following to evaluate the predictions:
```
python3 util/computePMSE.py 5 10
python3 util/computePMSE.py 10 10
python3 util/computePMSE.py 15 10
```
First argument specifies number of rounds of data (which allows the script
to locate the right directory inside ```generatedData``) while the second
argument specifies how many files are in that directory.
The results will hopefully match Table 2 from the paper.

## Bravery Game Experiments
Bravery game experiments are arguably the most trivial of all and 
consist of two simple experiments:

1. The first experiment sets parameters of agents to reflect the 
psychological game model, but varies beliefs of player 1 and investigates
whether an equilibrium is reached. 
To run, execute
```
webppl examples/bravery/src/simulations.wppl --require . --require examples/bravery -- --experiment 1 --log 1
```
The result will be in the form of three traces (for different values of 
initial beliefs) - these have to be inspected manually. The paper makes
a claim that an equilibrium is reached regardless of initial belief -
this will be confirmed if the traces are dominated by *bold* actions
from some point onwards. 

2. The second experiment studies how p1's lookahead affects the total
reward in the game. To run it, execute:

```
webppl examples/bravery/src/simulations.wppl --require . --require examples/bravery -- --experiment 2 --runs 5 --log 1
```
This prints total rewards of players accumulated over 10 rounds for three
different values of lookahead (1,3,5), averaged over five runs. The results
should roughly match the values in Table 3 (it used to be 'Table 4', fixed
on 18/05).

BONUS (added 18/05)
I haven't originally included anything about tic-tac-toe, since it's a
running example rather than an experiment. But I will briefly describe 
below how to use the tool to compute the values in Table 1.

It is very easy to confirm the values in Table 1b. It suffices to run
```
webppl examples/tictactoe/src/experiments.wppl --require . --require examples/tictactoe/ -- --experiment 1 --scenario 1
```

An appropriate scenario has been defined in ```experiments.wppl``` with
exactly the setup described in the paper. The simulation starts in state 
called s0 in the paper and only predicts the first move of the parent.
Running the above command will print the setup, followed by expected
utilities computed by the parent in state s0 and their computed action.
The expected utilities will hopefully match the values in Table 1b

Confirming values from Table 1a is a little more complicated than that. 
By default, the tool does not print these values as they are too 
"low-level" - if the tool were to print all such values, there would
be a lot of output. However, we can have the tool print exactly
the values we are interested in by modifying the expected utility 
function. It is hacky, but it works and is sometimes useful for debugging
purposes. In the future I would like to develop a more robust mechanism
for printing various debugging information, but for now this is what I have.
Anyway, to confirm the U values from the top row of Table 1a, one must
uncomment the following expression
```
(selfId === 1 && ofAgentID === 1 && _.isEqual(state, [['O',7],['X',0],['O',8],['X',4]])) ||
```
in the ```cognitiveAgent.wppl``` file, in ```expectedUtility``` function
(line 524). I have prepared this line specifically, normally it would
not be there... 
Now, after running the same commandline as before, the tool will 
additionally print the required expected utilities. The printed values
will hopefully match those in the Table, but note that action naming 
in the tool is different than in the paper: in the tool, squares of 
tic-tac-toe are identified by consecutive integers, as given below:

0 | 1 | 2
3 | 4 | 5
6 | 7 | 8

*Now that I was checking it, there's a small error in Table 1a, one of
-0.17 (the one at a_3^k) should be swapped with one of 1.09 (the one 
at a_5^k) - sorry for that!*

Finally, to confirm the second row of Table 1a, the one specifying 
probabilities, we will use the same hack, this time commenting out
an expression
```
(selfId === 1 && ofAgentID === 0 && _.isEqual(state, [['O',7],['X',0],['O',8],['X',4]])) ||
```
which is just below the previous one (line 525).

Again, this will print some additional expected utilities, this time 
utilities of the kid, computed by the parent in state called s1 in the 
paper for various actions of the kid. Based on those expected utilities,
using softmax choice formula (Eq 5 of decision making equations), one 
can compute the probabilities from the second row of Table 1a. 

I hope the above makes sense, I'm happy to explain more in case it 
doesn't. Finally, let me just say that I've made many changes to the
tool, in particular to do with running experiments, just before 
submitting the artifact. Therefore, many experiments that are
not described in this README may not work as they haven't been 
adapted to the new conventions. If the reviewers would like to 
experiment with some of those experiments, I'd be happy to update them
as needed. 
  