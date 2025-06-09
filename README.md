# TCL_workshop_vsd
## Requirements 
- Linux OS
- Yosys Synthesis Suite
- OpenTimer STA Tool
- TCL Matric Package

## Day 1 - Introduction to TCL and VSDSYNTH Toolbox Usage
Day 1's task is to create a command (vsdsynth) and pass a .csv file from the UNIX shell to the TCL script, taking into consideration mainly three general scenarios from the user's point of view.
![image](https://github.com/user-attachments/assets/2943e8c8-5e0a-4570-b611-07c34ad6d2c6)

### Input files provided in the directory 
![Screenshot 2025-06-08 231247](https://github.com/user-attachments/assets/2c0648e8-9486-4bf4-8cdd-46c35c60dcb3)

### openMSP430_design_details.csv
![Screenshot 2025-06-08 230832](https://github.com/user-attachments/assets/d0dcbe66-d50f-412b-a4b6-20b85cc23d0e)

## Implementation 
Task is to create _vsdsynth_ and _vsdsynth.tcl_ files.
The basic structure of bash code used for the implementation of general scenarios is shown below.
````
if ($#argv != 1) then
	echo "Info: Please provide the csv file"
	exit 1
endif

if (! -f $argv[1] || $argv[1] == "-help") then
	if ($argv[1] != "-help") then
		echo "Error: Cannot find csv file $argv[1]. Exiting..."
		exit 1
	else
		echo USAGE: ./vsdsynth \<csv file\>
		echo
		echo        where \csv file\> consists of 2 columns, below keyword being in 1st column and is Case Sensitive. PLease request PS for sample csv file
		echo
		echo        \<Design Name\> is the name of the top level module
                echo
                echo        \<Output Directory\> is the name of the output directory where you want to dump synthesis script, synthesized netlist and timing reports
                echo
                echo        \<Netlist Directory\> is the name of  directory where all the RTL netlist are present
                echo
                echo        \<Early Library Path\> is the file path of the early cell library to be used for STA
                echo
                echo        \<Late Library Path\> is the file path of the late cell library to be used for STA
                echo
                echo        \<Constraints file\> is csv file path of contraints to be used for STA
		echo
		exit 1
	endif
else tclsh ./vsdsynth.tcl $argv[1]
endif
````
