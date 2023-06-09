# GPSR_NLP

## Overview
NLP takes command by two ways (text-mode and Speech-mode), both can converted commands into string and feed to RASA(https://github.com/MehreenNaeem/GPSR_RASA). RASA is a platform that sort the sentence's words into their defined categories. It consist of vocabulary of words with their examples. After interpretation, NLP send the sorted dated to CRAM (https://github.com/MehreenNaeem/CRAM-GPSR). CRAM fill the parameters of plan with information and its knowledege. It also returns the results of plan to the NLP so that it will decide what to do next.

## Dependencies
### Setup new RASA enviorment 
install rasa via https://rasa.com/docs/rasa/installation/installing-rasa-open-source/
then make directory: 
```
mkdir my_rasa
```
go into the directory and initialize it
```
rasa init
rasa train
```
(NOTE: if you install rasa newly, it does not contain the related data. So after installation clone the following directory https://github.com/MehreenNaeem/GPSR_RASA. and do training so that model builds i.e it creates models folder and save trained models in it)
### In AlienWare (for ROBOcup team)
I have install rasa in venv. i.e @home /rasa_rb. Activate the venv enviornment
```
source ./venv/bin/activate
cd rasa_rb
```

run the rasa
```rasa run --enable-api```

### If you want to add words in vocabulary
checkout following https://github.com/MehreenNaeem/GPSR_RASA

### Install python dependencies
Whispher:
```pip install -U openai-whisper```

Natural Language Toolkit :
```sudo apt install python3-nltk ```
Downlaod the toolkit https://www.youtube.com/watch?v=81_WapLxt2w. open the python in terminal and do following
```
home$ python
>>>import nltk
>>>nltk.download()
```

PyDub :
```pip install pydub```

Play Sound :
```pip install playsound```

Speech Recognition :
```pip install SpeechRecognition```

Gtts :
```pip install gTTS```

PyAudio :
```sudo apt install python3-pyaudio```

### Audio drivers for HSR microphone
https://github.com/ros-drivers/audio_common

## How to Run GPSR_NLP and perform gpsr task
### RASA 
run the rasa


### CRAM 
On Alienware
```
 roslaunch suturo_demos suturo_bringup.launch upload_hsrb:=true use_rviz:=true
 roslaunch suturo_manipulation start_manipulation_easy.launch
```
Run the **Robokudo** 
```
[PHD][HSR]$ rosrun robokudo main.py _ae=my_demo _ros_pkg=human_perception
```
load the package in the REPL (start the REPL with $ roslisp_repl):
```
CL-USER> (swank:operate-on-system-for-emacs "suturo-demos" (quote load-op)) (in-package :su-demos) (swank:operate-on-system-for-emacs "suturo-real-hsr-pm" (quote load-op))
SU-DEMOS> (roslisp-utilities:startup-ros)
```
Listen the topics from the python scripts
```
SU-DEMOS>  (gpsr-subcribers)
```

### Launch NLP python script
In Aleinware, phd_workspaces/nlp_ws/src/gpsr_robocup package is used to launch the nlp scripts. source the workspace (~/phd_workspaces/nlp_ws [PHD][HSR]$ source devel/setup.bash) and then do following steps 
#### (Speech Mode)
Launch the **HSR microphone** (see below)
Go into the workspace and launch gpsr_robocup package (if you want to give command by audio)
```
roslaunch gpsr_robocup launch_all.xml
```
OR

```
gpsr_robocup$ rosrun gpsr_robocup Subcriber_cram_msg.py
gpsr_robocup$ rosrun gpsr_robocup Mainaudiowhisper.py
```
First enter the number of commands you want to perfrom (e.g Stage1 its 3) and type **"start"** to start the prcoess. Then it go to the starting position. When Speech mode is activated it ask you for the command and then confirm it by asking "is it right or not?". if you say "yes" it will excute it and if "no" it asks you to type the command on terminal.
(NOTE: if you want to specify the number of commands, goto gpsr_robocup/scripts/Mainaudiowhisper.py and set variable ``` gpsrstage1  = 3 ``` to that number)

#### (Text Mode) 
Go into the directory gpsr_robocup and in separate terminals, run the following nodes
```
gpsr_robocup$ rosrun gpsr_robocup Subcriber_cram_msg.py 
gpsr_robocup$ rosrun gpsr_robocup testingf.py
```
After running testingf node, it asks you for input in the terminal. First enter number of commands you want to excute in the whole demo e.g for gpsr-stage2 itss 5. Then it ask to type start to start the process and as a result it go to the starting position. Then ask to type the task. e.g
```
Enter the Task 1 :Can you please give me the milk
```
## Some Stuff related to Cram
GPSR-Robocup is depended on suturo-demo package. Following are the files that are related to it

**gpsr-ln.lisp** (listen the data from python script).
**gpsr-plans.lisp** (plans are defined in it).
**gpsr-knowledge.lisp** (knowledge about poses and keywords).
**gpsr-pub.lisp** (publish the data about plans).


So the data send in a formate of string array which has following slot division:

[plan_name object_name object_type person_name person_type attribute person_action color number location1 location2 room1 room2]

e.g 1) "provide an apple to the robert sitting in a bedroom"
```['deliver', 'apple', 'nil', 'robert', 'nil', 'nil', 'sitting', 'nil', 'nil', 'nil', 'nil', 'bedroom', 'nil']```

e.g 2) "transport two red object from desk in a bedroom to the dishwasher"
```['transport', 'object', 'nil', 'nil', 'nil', 'nil', 'nil', 'red', 'two', 'desk', 'dishwasher', 'bedroom', 'nil']```

e.g 3) "how many people in the dining room are boys"
```['count', 'nil', 'nil', 'nil', 'boys', 'nil', 'nil', 'nil', 'nil', 'nil', 'nil', 'room', 'nil']```


If the task is consist of more then one sentences e.g "find the bowl in the kitchen and give it to me", nlp divide it into two sentences i.e. "find the bowl in the kitchen" / "give it to me" and give them one by one to CRAM.
The gpsr-ln.lisp receives the data in the given string formate and save them in respective variables. then it match with the plans list and take variable according to the selected plan.
Some of the sentences may carry **pronouns** e.g in above command "give it to me". So "it" is supposed to be an object for this we need **buffer knowledge**. In gpsr_ln following buffer variables are saved as a global variable such that **previous-objectname**
### add plans in CRAM
if you want to add plans:
- define them in gpsr-plans.lisp using defun
- add plan title in list-of-plans and call plan function in subcriber-callback-function in gpsr-ln.lisp. e.g.
``` lisp = 
		 (when (eq *plan* :search)
		 	(print "Performing searching ...")
			(setf ?output (searching-object (object-to-be *objectname*))) ;; *objectname* = get-object-cram-name(?nlp-object-name)
			(print "searching Plan Done ...")
			(cram-talker ?output)
			)
```
here plan title is SEARCH and plan-function is searching-object (**Note** that plan title is the name of intent defined in RASA files data/nlu.yml and domain.yml). cram-talker send the plan name to the subcriber of GPSR-NLP after plan succeeded. plan function output the "plan name" when it succeeded and "fail" when fails. e.g

``` lisp = 
(defun searching-object (?object)
 (setf *perceived-object* nil)
(call-text-to-speech-action "Trying to perceive object")
 (let* ((possible-look-directions `(,*forward-upward*
                                     ,*left-downward*
                                     ,*right-upward*
                                     ,*forward-downward*))
         (?looking-direction (first possible-look-directions)))
    (setf possible-look-directions (cdr possible-look-directions))
	(cpl:with-failure-handling
			  (((or common-fail:perception-object-not-found
			  					desig:designator-error) (e)
					 (when possible-look-directions
					 (roslisp:ros-warn  (perception-failure) "Searching messed up: ~a~%Retring by turning head..." e)
				     (setf ?looking-direction (first possible-look-directions))
				     (setf possible-look-directions (cdr possible-look-directions))
				     (exe:perform (desig:an action 
				                           (type looking)
				                           (target (desig:a location
				                                            (pose ?looking-direction)))))
				     (cpl:retry))
				     
					 (roslisp:ros-warn (pp-plans pick-up) "No more retries left..... going back ")
					 (return-from searching-object "fail") ;;;; return fail when there is no choice left
				                
			 ))

                (setf *perceived-object* (exe:perform (desig:an action
						       (type detecting)
						       (object (desig:an object
						       (type ?object))))))
						       (call-text-to-speech-action "Successfully perceived object")
						       (return-from searching-object "search")))) ;;;; return plan name when it finish
```				    
All plans take input in the form of **keywords** which are get resolved by the **gpsr-knowledge.lisp**. If you want to add any function or variables on which plans depends, please define them in gpsr-knowledge

## HSR microphone
The package audio_common can stream audio and play it on another device
To start the streaming launch on HSR desktop. first source the workspace in hsr terminal. Open the terminal(ctrl+alt+T)

```
source /home/hsr-user/custom_controller_ws/devel/setup.bash
roslaunch audio_capture capture_wave.launch
```
The audio is streamed on the topic /audio/audio.

## TroubleShooting
### HSR Microphone
sometimes while running Speech mode HSR may not listen to the commands and print some weird things e.g
```
start recording...
recording stop...
66
 Thank you.
start recording...
recording stop...
26
 Thanks for watching.
 ```
 if values is less than hundred its means HSR mircophone died, So restart the nodes in HSR. so normal output look like this
```
 start recording...
recording stop...
1221
 navigate to the kitchen.
start recording...
recording stop...
264
 Yes.
``
(NOTE: if it faces these type of error, for backup it shifted to text mode)
 

