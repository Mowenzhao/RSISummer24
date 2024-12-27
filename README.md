# RSISummer24
Summer Project 2024 in Gluscevic Group. Modifications of CLASS RSI module: To make it compatible with python3, edit user interface, and add cosmological parameters. Email mowenzha@usc.edu for any confusion. 

## Installing CLASS
Follow the main instruction at https://github.com/lesgourg/class_public/tree/master for installing CLASS. 
RealSpaceInterface (RSI) is located at class_public/external/RealSpaceInterface.

## Before Running RSI
Follow the readme instruction of RSI: run pip install -r requirements.txt
run pip install opencv-python

Clear cache in RealSpaceInterface by rm cache/* or manually deleting the files

In class_public/source/perturbations.c

Comment out the line: 

if (ppt->has_source_delta_m == _TRUE_) {
       ppw->delta_m += 3. *ppw->pvecback[pba->index_bg_a]*ppw->pvecback[pba->index_bg_H] * ppw->theta_m/k2;

Line 889 states the reason: "You requested a very high z_pk=%e, higher than z_rec=%e. This works very well when you ask only transfer functions, e.g. with 'output=mTk' or 'output=mTk,vTk'. But if you need the total matter (e.g. with 'mPk', 'dCl', etc.) there is an issue with the calculation of delta_m at very early times. By default, delta_m is a gauge-invariant variable (the density fluctuation in comoving gauge) and this quantity is hard to get accurately at very early times. The solution is to define delta_m as the density fluctuation in the current gauge, synchronous or newtonian. For the moment this must be done manually by commenting the line 'ppw->delta_m += 3. *ppw->pvecback[pba->index_bg_a]*ppw->pvecback[pba->index_bg_H] * ppw->theta_m/k2;' in perturbations_sources(). In the future there will be an option for doing it in an easier way."

### Changes already implemented to RSI to make it compatible with Python 3 
(official updates to be made in 2025.10)

Syntax change:
Floor division: / → //
File editing: “w” → “wb”; “r” → “rb”

Specifying dataencoding:
Calc2D/Database.py
 - In def __read_database(self):, return pickle.load(f) --> return pickle.load(f, encoding='latin1')
 - In def __getitem__(self, key):, return pickle.load(f) --> return pickle.load(f, encoding='latin1')
 - In def __write_database(self):, pickle.dump(self.db, f) --> pickle.dump(self.db, f, protocol=pickle.HIGHEST_PROTOCOL)

## Running RSI
In your terminal:
Navigate to class_public/external/RealSpaceInteraface
run python tornadoserver.py 
(if interested in a more instructive simulation, run python tornadoserver-annotated.py)
It should give you a link of the localhost, which direct you to the graphic user interface.

If at any point the program seems stuck, check if there are any error messages or prompts in the terminal. 

### In the GUI

![Screenshot 2024-12-27 105954](https://github.com/user-attachments/assets/244bac92-79d6-47cc-a942-91d4afcbd863)


The control buttons in the control panel are organized in order of their use from up to down.

### Configure Redshift: 

You can define the range of your simulation in terms of z. The simulation is based on calculations iterated individual time intervals. You can cut the entire simulated time range into several segments, and for each segment you can customize the number of intervals and their distribution (logarithmic or linear).

There are two ways that you can set up your initial condition, Gaussian or Scale invariant, click on one of them to select, modify any distribution parameters, and click on **set initial condition** to initialize the simulation. A display of the initial data should appear. 

### Cosmological Parameters

Toggle the slider to adjust cosmological parameters;
Note that some values of parameters might not be physical and can result in calculation errors and simulation failure, in which case you will need to relaunch the program (python tornadoserver.py). I have not figured out a better way to solve this other than restricting the range of the parameters.

**Reset Parameters** returns the parameters to their default value

### Start Calculation

Calculation progress bar should appear.

Once the calculation is finished, you can view the calculated graphs on the left and the animated simulation by pressing the play button at the top. Dropdown menu at the top left allows you to choose which component (cdm, baryon, etc) to display. 

Changing the cosmological parameters without resetting the initial condition, then pressing **start calculations** again allows you to view simulations of different parameters side by side. 

The red **Simulations** button on top allows you to manage the existing simulations: displaying different components side by side, saving, recording, reloading parameters, etc. 

## General Workflow of RSI

The python file tornadoserver.py connects the frontend (GUI) and backend (CLASS).
In tornadoserver.py, there are two main functions: 
 - SimulationHandler: Request web and render initial GUI display
 - DataConnection: Receives three types of parameters from the user and performs actions accordingly

Initial condition: display the initial power density map

Cosmological parameters: send to CLASS for calculations

Start: propagate the transfer function through all steps of z.

**A far more detailed description can be found in the annotations of tornadoserver-annotated.py.**

## How to modify RSI

### Static displays
The html files in /templates define the general framework for the user interface. Such as the headers, plot windows, colormaps, and fonts.
The css files in /static/css specifies the parameters used in /templates and overrides the html files.

### Dynamic displays
The json files in /static/js and static/threejs define user interactions, such as the parameter input sliders and animations. 

### Cosmological parameters
In /static/js, files parameterConfig.js and paramGui.js defines the input and display of cosmological parameters, which through functions in tornadoserver.py get passed to files in /Calc2D to be calculated by CLASS.

Simply editing parameterConfig.js allows you to modify ranges of cosmological parameters and add extra parameters, either as specific values or additional sliders. paramGui.js has already streamlined the entire process. 
