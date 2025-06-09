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
In the command vsdsynth, I have implemented different scenarios from user's point of view in the bash script.
![Screenshot 2025-06-08 231833](https://github.com/user-attachments/assets/3165fa32-dc32-44cc-ae61-c354ddd452ac)
In the above picture we can see scenarios like giving no input file to the command, giving wrong number of inputs to the command, giving wrong .csv file and asking help to know what the command can do and what the command needs.

## Day 2 - Variable Creation and Processing Constraints from CSV
Day 2's task is to basically write the TCL code in vsdsynth.tcl for variable creation, file/directory existence check, and the processing of the constraints csv file to convert it into format[1] (which is the constraints format taken as input by Yosys tool) as well as into SDC (Synopsys Design Constraints) format (which is the industry standard format).
![image](https://github.com/user-attachments/assets/b70a3055-7f25-4b97-944f-1f81b9d903d5)

Now let us see what's inside the openMSP430_design_constraints.csv file 

![Screenshot 2025-06-08 231922](https://github.com/user-attachments/assets/7c656855-f25d-478c-bdbc-574ab6b31a5e)

### Implementation 
**vsdsynth.tcl** (using matric package and creating a mtric from the details.csv file)
''''
set filename [lindex $argv 0]
package require csv
package require struct::matrix
struct::matrix m
set f [open $filename]
csv::read2matrix $f m , auto
close $f
set columns [m columns]
#m add columns $columns
m link my_arr
set num_of_rows [m rows]
````
 **Variable creation**
I have auto-cretaed variables by converting the .csv file into a matric and calculating the number of rows and columns and creating variables for each block as my_varr.
````
set i 0
	while {$i < $num_of_rows} {
		puts "\nInfo: Setting $my_arr(0,$i) as '$my_arr(1,$i)'"
		if {$i == 0} {
			set [string map {" " ""} $my_arr(0,$i)] $my_arr(1,$i)
		} else {
			set [string map {" " ""} $my_arr(0,$i)] [file normalize $my_arr(1,$i)]
		}
		set i [expr {$i+1}]
	}
}

puts "\nInfo: Below are the list of initial variables and their values. User can use these variables for further debug. Use 'puts <variable name>' command to query value of below variables"
puts "DesignName = $DesignName"
puts "OutputDirectory = $OutputDirectory"
puts "NetlistDirectory = $NetlistDirectory"
puts "EarlyLibraryPath = $EarlyLibraryPath"
puts "LateLibraryPath = $LateLibraryPath"
puts "ConstraintsFile = $ConstraintsFile"
````
Converted the names into variable by removing the space between them and assigned the paths to the variables.
_Screenshot_

![Screenshot 2025-06-08 232134](https://github.com/user-attachments/assets/e3e060f0-c91e-4bbc-adfe-3c3e2bfa7219)

### File/Directory Existence Check

I have written the code to check the existence of all files and directories wherein the program exits in case they are not found since their existence is crucial for the program to move further except for the output directory, which is created if it does not exist. The basic code of the same and screenshots of the terminal demonstrating the functionality, namely one showing the creation of a new output directory and another in which an output directory exists but a constraints file does not exist, are shown below.

_Code_
````
if {![file isdirectory $OutputDirectory]} {
	puts "\nInfo: Cannot find output directory $OutputDirectory. Creating $OutputDirectory"
	file mkdir $OutputDirectory
} else {
	puts "\nInfo: Output directory found in path $OutputDirectory"
}
if {![file isdirectory $NetlistDirectory]} {
        puts "\nInfo: Cannot find RTL netlist directory $NetlistDirectory. Exiting..."
        exit
} else {
        puts "\nInfo: RTL Netlist directory found in path $NetlistDirectory"
}
if {![file exists $EarlyLibraryPath]} {
        puts "\nInfo: Cannot find early cell library in path $EarlyLibraryPath. Exiting..."
        exit
} else {
        puts "\nInfo: Early cell library found in path $EarlyLibraryPath"
}
if {![file exists $LateLibraryPath]} {
        puts "\nInfo: Cannot find late cell library in path $LateLibraryPath. Exiting..."
        exit
} else {
        puts "\nInfo: Late cell library found in path $LateLibraryPath"
}
if {![file exists $ConstraintsFile]} {
        puts "\nInfo: Cannot find constraints file in path $ConstraintsFile. Exiting..."
        exit
} else {
        puts "\nInfo: Contraints file found in path $ConstraintsFile"
}
````
_Screenshots_

![Screenshot 2025-06-08 232350](https://github.com/user-attachments/assets/c5e65508-b28e-41b6-98b8-7ccf5c655098)

![Screenshot 2025-06-08 232440](https://github.com/user-attachments/assets/91074eff-d24e-41b9-8056-18d542dc173a)

### Processing of the constraints openMSP430_design_constraints.csv file

The file was successfully processed and converted into a matrix, and the rows and columns count were extracted, as well as the starting rows of clocks, inputs, and outputs. The basic code of the same and a screenshot of the terminal with several "puts" printing out the variables are shown below.

_Code_
````
puts "\nInfo: Dumping SDC contraints file for $DesignName"
::struct::matrix constraints
set  chan [open $ConstraintsFile]
csv::read2matrix $chan constraints  , auto
close $chan
set number_of_rows [constraints rows]
puts "number_of_rows are $number_of_rows"
set number_of_columns [constraints columns]
puts "number_of_columns are $number_of_columns"
set clock_start [lindex [lindex [constraints search all CLOCKS] 0 ] 1]
puts "clock_start = $clock_start"
set clock_start_column [lindex [lindex [constraints search all CLOCKS] 0 ] 0]
puts "clock_start_column = $clock_start_column"
#----check row number for "inputs" section in constraints.csv------------#
set input_ports_start [lindex [lindex [constraints search all INPUTS] 0 ] 1]
puts "input_ports_start = $input_ports_start"
#----check row number for "inputs" section in constraints.csv------------#
set output_ports_start [lindex [lindex [constraints search all OUTPUTS] 0 ] 1]
puts "output_ports_start = $output_ports_start"
````

_Screenshot_

![Screenshot 2025-06-08 232624](https://github.com/user-attachments/assets/9694a3d2-a25c-4650-80b3-545522a0304f)

## Day 3 - Processing Clock and Input Constraints from CSV and dumping SDC






























