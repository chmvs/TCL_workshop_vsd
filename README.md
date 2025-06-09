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
````
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
Day 3's task is to basically process constraints in a csv file for clocks and inputs and dump SDC commands into a .sdc file with actual processed data. It involves several matrix search algorithms and also an algorithm to identify inputs that are buses and bits differently.

## Implementation

I have successfully completed Day 3 tasks, namely processing constraints in a csv file for clocks and inputs and dumping SDC commands into a .sdc file with actual processed data.

**Processing of the constraints .csv file for CLOCKS and dumping SDC commands to .sdc**
I have successfully processed the csv file for CLOCKS data and dumped clock-based SDC commands to .sdc file. The basic code of the same and screenshots of the terminal with several "puts" printing out the variables and user debug information as well as output .sdc are shown below.

_Code_
````
#----------------------clock constraints-------------------------------------#
#----------------------clock latency constraints-----------------------------#

set clock_early_rise_delay_start [lindex [lindex [constraints search rect $clock_start_column $clock_start [expr {$number_of_columns-1}] [expr {$input_ports_start-1}] early_rise_delay] 0] 0]
set clock_early_fall_delay_start [lindex [lindex [constraints search rect $clock_start_column $clock_start [expr {$number_of_columns-1}] [expr {$input_ports_start-1}] early_fall_delay] 0] 0]
set clock_late_rise_delay_start [lindex [lindex [constraints search rect $clock_start_column $clock_start [expr {$number_of_columns-1}] [expr {$input_ports_start-1}] late_rise_delay] 0] 0]
set clock_late_fall_delay_start [lindex [lindex [constraints search rect $clock_start_column $clock_start [expr {$number_of_columns-1}] [expr {$input_ports_start-1}] late_fall_delay] 0] 0]
#----------------clock transition contraints---------------------------------#
set clock_early_rise_slew_start [lindex [lindex [constraints search rect $clock_start_column $clock_start [expr {$number_of_columns-1}] [expr {$input_ports_start-1}] early_rise_slew] 0] 0]
set clock_early_fall_slew_start [lindex [lindex [constraints search rect $clock_start_column $clock_start [expr {$number_of_columns-1}] [expr {$input_ports_start-1}] early_fall_slew] 0] 0]
set clock_late_rise_slew_start [lindex [lindex [constraints search rect $clock_start_column $clock_start [expr {$number_of_columns-1}] [expr {$input_ports_start-1}] late_rise_slew] 0] 0]
set clock_late_fall_slew_start [lindex [lindex [constraints search rect $clock_start_column $clock_start [expr {$number_of_columns-1}] [expr {$input_ports_start-1}] late_fall_slew] 0] 0]
set sdc_file [open $OutputDirectory/$DesignName.sdc "w"]
set i [expr {$clock_start+1}]
set end_of_ports [expr {$input_ports_start-1}]
puts "\nInfo-SDC: Working on clock constraints....."
while {$i < $end_of_ports} {
	puts "Working on clock [constraints get cell 0 $i]"
	puts -nonewline $sdc_file "\ncreate_clock -name [constraints get cell 0 $i] -period [constraints get cell 1 $i] -waveform \{0 [expr {[constraints get cell 1 $i]*[constraints get cell 2 $i]/100}]\} \[get_ports [constraints get cell 0 $i]\]"
	puts -nonewline $sdc_file "\nset_clock_transition -rise -min  [constraints get cell $clock_early_rise_slew_start $i] \[get_clocks [constraints get cell 0 $i]\]"
	puts -nonewline $sdc_file "\nset_clock_transition -fall -min  [constraints get cell $clock_early_fall_slew_start $i] \[get_clocks [constraints get cell 0 $i]\]"
	puts -nonewline $sdc_file "\nset_clock_transition -rise -max  [constraints get cell $clock_late_rise_slew_start $i] \[get_clocks [constraints get cell 0 $i]\]"
	puts -nonewline $sdc_file "\nset_clock_transition -fall -max  [constraints get cell $clock_late_fall_slew_start $i] \[get_clocks [constraints get cell 0 $i]\]"
	puts -nonewline $sdc_file "\nset_clock_latency -source -early -rise  [constraints get cell $clock_early_rise_delay_start $i] \[get_clocks [constraints get cell 0 $i]\]"
	puts -nonewline $sdc_file "\nset_clock_latency -source -early -fall  [constraints get cell $clock_early_fall_delay_start $i] \[get_clocks [constraints get cell 0 $i]\]"
	puts -nonewline $sdc_file "\nset_clock_latency -source -late -rise  [constraints get cell $clock_late_rise_delay_start $i] \[get_clocks [constraints get cell 0 $i]\]"
	puts -nonewline $sdc_file "\nset_clock_latency -source -late -fall  [constraints get cell $clock_late_fall_delay_start $i] \[get_clocks [constraints get cell 0 $i]\]"
	
	set i [expr {$i+1}]
}
````
_Screenshot_

![Screenshot 2025-06-08 232947](https://github.com/user-attachments/assets/ba3f11e3-dc9a-48e3-bd19-35c49be0b406)

_openMSP430.sdc_

![Screenshot 2025-06-08 233015](https://github.com/user-attachments/assets/7871f497-4adb-4ac3-bd95-d5d08db470eb)

**Processing of the constraints .csv file for INPUTS and dumping SDC commands to .sdc**

I have successfully processed the csv file for INPUTS data as well as differentiated bit and bus inputs and dumped input-based SDC commands to .sdc file. The basic code of the same and screenshots of the terminal with several "puts" printing out the variables and user debug information as well as output .sdc are shown below.

_Code_
````
#--------------------input constraints-------------------------------#
set input_early_rise_delay_start [lindex [lindex [constraints search rect $clock_start_column $input_ports_start [expr {$number_of_columns-1}] [expr {$output_ports_start-1}] early_rise_delay] 0] 0]
set input_early_fall_delay_start [lindex [lindex [constraints search rect $clock_start_column $input_ports_start [expr {$number_of_columns-1}] [expr {$output_ports_start-1}] early_fall_delay] 0] 0]
set input_late_rise_delay_start [lindex [lindex [constraints search rect $clock_start_column $input_ports_start [expr {$number_of_columns-1}] [expr {$output_ports_start-1}] late_rise_delay] 0] 0]
set input_late_fall_delay_start [lindex [lindex [constraints search rect $clock_start_column $input_ports_start [expr {$number_of_columns-1}] [expr {$output_ports_start-1}] late_fall_delay] 0] 0]

set input_early_rise_slew_start [lindex [lindex [constraints search rect $clock_start_column $input_ports_start [expr {$number_of_columns-1}] [expr {$output_ports_start-1}] early_rise_slew] 0] 0]
set input_early_fall_slew_start [lindex [lindex [constraints search rect $clock_start_column $input_ports_start [expr {$number_of_columns-1}] [expr {$output_ports_start-1}] early_fall_slew] 0] 0]
set input_late_rise_slew_start [lindex [lindex [constraints search rect $clock_start_column $input_ports_start [expr {$number_of_columns-1}] [expr {$output_ports_start-1}] late_rise_slew] 0] 0]
set input_late_fall_slew_start [lindex [lindex [constraints search rect $clock_start_column $input_ports_start [expr {$number_of_columns-1}] [expr {$output_ports_start-1}] late_fall_slew] 0] 0]

set related_clock [lindex [lindex [constraints search rect $clock_start_column $input_ports_start [expr {$number_of_columns-1}] [expr {$output_ports_start-1}] clocks] 0 ] 0]
set i [expr {$input_ports_start+1}]
set end_of_ports [expr {$output_ports_start-1}]
puts "\nInfo-SDC: WOrking on IO constraints......."
puts "\nInfo-SDC: Categorizing input ports as bits and bussed"

while {$i < $end_of_ports} {
#------------------------------------optional----------differentiating input ports as bussed and bits-------------------#
set netlist [glob -dir $NetlistDirectory *.v]
set tmp_file [open /tmp/1 w]
foreach f $netlist {
	set fd [open $f]
	puts "reading file $f"
	while {[gets $fd line] != -1} {
		set pattern1 " [constraints get cell 0 $i];"
		if {[regexp -all -- $pattern1 $line]} {
			puts "pattern1 \"$pattern1\" found and matching in verilog file \"$f\" is \"$line\""
			set pattern2 [lindex [split $line ";"] 0]
			puts "creatng pattern2 by splitting pattern1 using semi-colon as delimiter => \"$pattern2\""
			if {[regexp -all {input} [lindex [split $pattern2 "\S+"] 0]]} {
				puts "out of all patterns, \"$pattern2\"has matching string \"input\". Sp preserving this line and ignoring others"
				set s1 "[lindex [split $pattern2 "\S+"] 0] [lindex [split $pattern2 "\S+"] 1] [lindex [split $pattern2 "\S+"] 2]"
				puts "printing first 3 elements of pattern2 as \"$s1\" using space as delimiter"
				puts -nonewline $tmp_file "\n[regsub -all {\s+} $s1 " "]"
				puts "replace mutiple spaces in s1 by single space and reformant as \"[regsub -all {\s+} $s1 " "]\""
			        }
			}
		
		}
	

close $fd
}
close $tmp_file

set tmp_file [open /tmp/1 r]
#puts "reading [read $tmp_file]"
#puts "reading /tmp/1 file as [split [read $tmp_file] \n]"
#puts "sorting /tmp/1 contents as [lsort -unique [split [read $tmp_file] \n ]]"
#puts "joining /tmp/1 as [join [lsort -unique [split [read $tmp_file] \n ]] \n]"
set tmp2_file [open /tmp/2 w]
puts -nonewline $tmp2_file "[join [lsort -unique [split [read $tmp_file] \n]] \n]"
close $tmp_file
close $tmp2_file

set tmp2_file [open /tmp/2 r]
#puts "Count is [llength [read $tmp2_file]]"
set count [llength [read $tmp2_file]]
#puts "Splitting content of tmp_2 using space and counting number of elements as $count"

if {$count > 2} {
	set inp_ports [concat [constraints get cell 0 $i]*]
	puts "bussed"
} else {
	set inp_ports [constraints get cell 0 $i]
	puts "not bussed"
}
	puts "input port name is $inp_ports since count is $count\n"
	puts -nonewline $sdc_file "\nset_input_delay -clock \[get_clocks [constraints get  cell $related_clock $i]\] -min -rise -source_latency_included [constraints get cell $input_early_rise_delay_start $i] \[get_ports $inp_ports\]"
	puts -nonewline $sdc_file "\nset_input_delay -clock \[get_clocks [constraints get  cell $related_clock $i]\] -min -fall -source_latency_included [constraints get cell $input_early_fall_delay_start $i] \[get_ports $inp_ports\]"
	puts -nonewline $sdc_file "\nset_input_delay -clock \[get_clocks [constraints get  cell $related_clock $i]\] -max -rise -source_latency_included [constraints get cell $input_late_rise_delay_start $i] \[get_ports $inp_ports\]"
	puts -nonewline $sdc_file "\nset_input_delay -clock \[get_clocks [constraints get  cell $related_clock $i]\] -max -fall -source_latency_included [constraints get cell $input_late_fall_delay_start $i] \[get_ports $inp_ports\]"

	puts -nonewline $sdc_file "\nset_input_transition -clock \[get_clocks [constraints get  cell $related_clock $i]\] -min -rise -source_latency_included [constraints get cell $input_early_rise_slew_start $i] \[get_ports $inp_ports\]"
        puts -nonewline $sdc_file "\nset_input_transition -clock \[get_clocks [constraints get  cell $related_clock $i]\] -min -fall -source_latency_included [constraints get cell $input_early_fall_slew_start $i] \[get_ports $inp_ports\]"
        puts -nonewline $sdc_file "\nset_input_transition -clock \[get_clocks [constraints get  cell $related_clock $i]\] -max -rise -source_latency_included [constraints get cell $input_late_rise_slew_start $i] \[get_ports $inp_ports\]"
        puts -nonewline $sdc_file "\nset_input_transition -clock \[get_clocks [constraints get  cell $related_clock $i]\] -max -fall -source_latency_included [constraints get cell $input_late_fall_slew_start $i] \[get_ports $inp_ports\]"

	set i [expr {$i+1}]
}
close $tmp2_file
````

_Screenshots_

![Screenshot 2025-06-08 233601](https://github.com/user-attachments/assets/64c2c5d9-16ed-47b6-b42b-f5a6263daf79)

![Screenshot 2025-06-08 233405](https://github.com/user-attachments/assets/d5f99c77-0937-4632-8fad-61dacd945bcb)

_openMSP430.sdc_

![Screenshot 2025-06-08 233324](https://github.com/user-attachments/assets/13a0990d-00fc-4c23-a4c8-abc1a8db401e)

## Day 4 - Complete Scripting and Yosys Synthesis Introduction

Day 4's tasks included the output section processing and dumping of the SDC file, sample Yosys synthesis using example memory and explanation, Yosys hierarchy check, and its error handling.

### Implementation 

I have successfully completed Day 4 tasks, namely processing constraints csv file for outputs and dumping SDC commands to .sdc file with actual processed data; learning sample memory synthesis and its memory write and read processes; dumping the hierarchy check Yosys script; and writing code handling errors in hierarchy check.

**Processing of the constraints .csv file for OUTPUTS and dumping SDC commands to .sdc**

I have successfully processed the csv file for OUTPUTS data as well as differentiated bit and bus outputs and dumped output-based SDC commands to .sdc file. The basic code of the same and screenshots of the terminal with several "puts" printing out the variables and user debug information as well as output .sdc are shown below.

_Code_
````
set output_early_rise_delay_start [lindex [lindex [constraints search rect $clock_start_column $output_ports_start [expr {$number_of_columns-1}] [expr {$number_of_rows-1}] early_rise_delay] 0] 0]
set output_early_fall_delay_start [lindex [lindex [constraints search rect $clock_start_column $output_ports_start [expr {$number_of_columns-1}] [expr {$number_of_rows-1}] early_fall_delay] 0] 0]
set output_late_rise_delay_start [lindex [lindex [constraints search rect $clock_start_column $output_ports_start [expr {$number_of_columns-1}] [expr {$number_of_rows-1}] late_rise_delay] 0] 0]
set output_late_fall_delay_start [lindex [lindex [constraints search rect $clock_start_column $output_ports_start [expr {$number_of_columns-1}] [expr {$number_of_rows-1}] late_fall_delay] 0] 0]
set output_load_start [lindex [lindex [constraints search rect $clock_start_column $output_ports_start [expr {$number_of_columns-1}] [expr {$number_of_rows-1}] load] 0 ] 0]

set related_clock [lindex [lindex [constraints search rect $clock_start_column $output_ports_start [expr {$number_of_columns-1}] [expr {$number_of_rows-1}] clocks] 0 ] 0]
set i [expr {$output_ports_start+1}]
set end_of_ports [expr {$number_of_rows-1}]
puts "\nInfo-SDC: Working on IO constraints......."
puts "\nInfo-SDC: Categorizing output ports as bits and bussed"

while {$i < $end_of_ports} {
set netlist [glob -dir $NetlistDirectory *.v]
set tmp_file [open /tmp/1 w]
foreach f $netlist {
        set fd [open $f]
        #puts "reading file $f"
        while {[gets $fd line] != -1} {
                set pattern1 " [constraints get cell 0 $i];"
                if {[regexp -all -- $pattern1 $line]} {
                        #puts "pattern1 \"$pattern1\" found and matching in verilog file \"$f\" is \"$line\""
                        set pattern2 [lindex [split $line ";"] 0]
                        #puts "creatng pattern2 by splitting pattern1 using semi-colon as delimiter => \"$pattern2\""
                        if {[regexp -all {input} [lindex [split $pattern2 "\S+"] 0]]} {
                                #puts "out of all patterns, \"$pattern2\"has matching string \"input\". Sp preserving this line and ignoring others"
                                set s1 "[lindex [split $pattern2 "\S+"] 0] [lindex [split $pattern2 "\S+"] 1] [lindex [split $pattern2 "\S+"] 2]"
                                #puts "printing first 3 elements of pattern2 as \"$s1\" using space as delimiter"
                                puts -nonewline $tmp_file "\n[regsub -all {\s+} $s1 " "]"
                                #puts "replace mutiple spaces in s1 by single space and reformant as \"[regsub -all {\s+} $s1 " "]\""
                                }
                        }

                }
close $fd
}
close $tmp_file

set tmp_file [open /tmp/1 r]
#puts "reading [read $tmp_file]"
#puts "reading /tmp/1 file as [split [read $tmp_file] \n]"
#puts "sorting /tmp/1 contents as [lsort -unique [split [read $tmp_file] \n ]]"
#puts "joining /tmp/1 as [join [lsort -unique [split [read $tmp_file] \n ]] \n]"
set tmp2_file [open /tmp/2 w]
puts -nonewline $tmp2_file "[join [lsort -unique [split [read $tmp_file] \n]] \n]"
close $tmp_file
close $tmp2_file

set tmp2_file [open /tmp/2 r]
#puts "Count is [llength [read $tmp2_file]]"
set count [llength [read $tmp2_file]]
#puts "Splitting content of tmp_2 using space and counting number of elements as $count"

if {$count > 2} {
        set op_ports [concat [constraints get cell 0 $i]*]
        #puts "bussed"
} else {
        set op_ports [constraints get cell 0 $i]
        #puts "not bussed"
}
        #puts "output port name is $op_ports since count is $count\n"
        puts -nonewline $sdc_file "\nset_output_delay -clock \[get_clocks [constraints get  cell $related_clock $i]\] -min -rise -source_latency_included [constraints get cell $output_early_rise_delay_start $i] \[get_ports $op_ports\]"
        puts -nonewline $sdc_file "\nset_output_delay -clock \[get_clocks [constraints get  cell $related_clock $i]\] -min -fall -source_latency_included [constraints get cell $output_early_fall_delay_start $i] \[get_ports $op_ports\]"
        puts -nonewline $sdc_file "\nset_output_delay -clock \[get_clocks [constraints get  cell $related_clock $i]\] -max -rise -source_latency_included [constraints get cell $output_late_rise_delay_start $i] \[get_ports $op_ports\]"
        puts -nonewline $sdc_file "\nset_output_delay -clock \[get_clocks [constraints get  cell $related_clock $i]\] -max -fall -source_latency_included [constraints get cell $output_late_fall_delay_start $i] \[get_ports $op_ports\]"
        puts -nonewline $sdc_file "\nset_load [constraints get cell $output_load_start $i] \[get_ports $op_ports\]"

        set i [expr {$i+1}]

}

close $tmp2_file
#return
close $sdc_file

puts "\nInfo: SDC created. PLease use constraints in path $OutputDirectory/$DesignName.sdc"
````

_Screenshots_

![Screenshot 2025-06-08 234038](https://github.com/user-attachments/assets/9ee8b574-0472-42a7-b8fd-3a213d230623)

_openMSP430.sdc_

![Screenshot 2025-06-08 234145](https://github.com/user-attachments/assets/6a3ae7c2-8ed6-49eb-a3bc-3c5a0d418194)

/tmp/1 and /tmp/2 files will be similar for input and output ports.

**Memory module yosys synthesis and explanation**
The verilog code memory.v for a single-bit address and single-bit data memory unit is given below.
_Code_
````
module memory (CLK, ADDR, DIN, DOUT);

parameter wordSize = 1;
parameter addressSize = 1;

input ADDR, CLK;
input [wordSize-1:0] DIN;
output reg [wordSize-1:0] DOUT;
reg [wordSize:0] mem [0:(1<<addressSize)-1];

always @(posedge CLK) begin
	mem[ADDR] <= DIN;
	DOUT <= mem[ADDR];
	end

endmodule
````

The basic Yosys script memory.ys to run this and obtain a gate-level netlist and 2D representation of the memory module in gate components is provided below.

_Script_
````
read_liberty -lib -ignore_miss_dir -setattr blackbox /home/kunalg/Desktop/work/openmsp430/openmsp430/osu018_stdcells.lib
read_verilog memory.v
synth top memory
splitnets -ports -format ___
dfflibmap -liberty /home/kunalg/Desktop/work/openmsp430/openmsp430/osu018_stdcells.lib
opt
abc -liberty /home/kunalg/Desktop/work/openmsp430/openmsp430/osu018_stdcells.lib
flatten
clean -purge
opt
clean
write_verilog memory_synth.v
````
The output view of netlist from the code is shown below.

![image](https://github.com/user-attachments/assets/f2efca83-216a-4a1d-b0d7-77a3547c3b9c)

Memory write process explained in following images using truth table

Basic illustration of the write process

![image](https://github.com/user-attachments/assets/7415d35b-18c5-4cfb-a0be-d8ba60237914)

Before first rising edge of the clock

![image](https://github.com/user-attachments/assets/d94283a0-a81d-4118-beea-6e27220f5e6b)

After first rising edge of the clock - write process done

![image](https://github.com/user-attachments/assets/801f8700-d7f5-4806-8a21-186ff24146d4)

Memory read process explained in following images using truth table

Basic illustration of the read process

![image](https://github.com/user-attachments/assets/1ac81e5c-fec2-4fa8-818b-1f77d248a41e)

After first rising edge and before second rising edge of the clock

![image](https://github.com/user-attachments/assets/46a00262-354e-4c26-af9c-baf2547a9d86)

After second rising edge of the clock - read process done

![image](https://github.com/user-attachments/assets/a941c306-abb1-479c-a205-878ebfe7917f)

**Hierachy Check Script**
I have successfully written the code for dumping the hierarchy check script. The basic code of the same and screenshots of the terminal with several "puts" printing out the variables and user debug information as well as output .hier.ys are shown below.

_Code_
````
#Hierarchy Check
puts "\nInfo: Creating hierarchy check script to be used by Yosys"
set data "read_liberty -lib -ignore_miss_dir -setattr blackbox ${LateLibraryPath}"
puts "data is \"$data\""
set filename "$DesignName.hier.ys"
puts "Filename is \"$filename\""
set fileId [open $OutputDirectory/$filename "w"]
puts -nonewline $fileId $dat
set netlist [glob -dir $NetlistDirectory *.v]
puts "Netlist is \"$netlist\""
foreach f $netlist {
	set data $f
	puts "Data is \"$f\""	
	puts -nonewline $fileId "\nread_verilog $f"
}
puts -nonewline $fileId "\nhierarchy -check"
close $fileId
````
_Screenshots_

![Screenshot 2025-06-08 234410](https://github.com/user-attachments/assets/1670c404-1b68-4a8a-8ca6-d441a91ec5c4)

_openMSP430.hier.ys_

![Screenshot 2025-06-08 234446](https://github.com/user-attachments/assets/03e206d2-cf7d-4e8b-86c9-8a219dcc3348)

**Hierarchy Check and Error Handling**
I have successfully written the code for hierarchy check error handling in case any error pops up during hierarchy check run in Yosys and exits if hierarchy check fails. The basic code of the same and screenshots of the terminal with several "puts" printing out the variables and user debug information are shown below.
_Code_
````
set my_err [catch {exec yosys -s $OutputDirectory/$DesignName.hier.ys >& $OutputDirectory/$DesignName.hierarchy_check.log} msg]
puts "error flag is $my_err"
if { $my_err } {
set filename "$OutputDirectory/$DesignName.hierarchy_check.log"
	puts "Log file name is $filename"
	set pattern {referenced in module}
	puts "Pattern is $pattern"
	set count 0
	set fid [open $filename r]
	while {[gets $fid line] != -1} {
		incr count [regexp -all -- $pattern $line]
		if {[regexp -all -- $pattern $line]} {
			puts "\nError: Module [lindex $line 2] is not part of design $DesignName. Please correct RTL in the path '$NetlistDirectory'"
			puts "\nInfo: Hierarchy check has FAILED"
		}
	}
	close $fid
} else {
	puts "\nInfo: Hierarchy check is PASS"
}
puts "\nInfo: PLease find hierarchy check details in [file normalize $OutputDirectory/$DesignName.hierarchy_check.log] for more info"
cd $working_dir
````
_Screenshots_

![Screenshot 2025-06-09 000100](https://github.com/user-attachments/assets/fe4f4e2b-1db8-424a-8f69-4eca9a8c001f)

_openMSP430.hierarchy_check.log_

![Screenshot 2025-06-09 000224](https://github.com/user-attachments/assets/e20368f3-3d6b-4dc4-bcc7-44de0a275504)

![Screenshot 2025-06-09 000250](https://github.com/user-attachments/assets/3c131b3c-409c-40df-a420-14077ed34b38)

## Day 5 - Advanced Scripting Techniques and Quality of Results (QoR) Generation
Day 5's tasks are to run main synthesis in Yosys, learn about procs and use them at the application level, create commands, and write necessary files required for the OpenTimer tool, such as .conf - .spef - .timing, write an OpenTimer script, run an OpenTimer STA, and collect the required data to form QoR from .results file generated from OpenTimer STA run and finally print the collected data in a tool-standard QoR output format.

### Implementation
I have successfully coded all the required elements to achieve Day 5 tasks, and all the details of the sub-tasks achieved are shown below.
**Main Yosys synthesis script dumping**
I have successfully written the code for the main Yosys synthesis script .ys file and dumped the script. The basic code of the same and screenshots of the terminal with several "puts" printing out the variables and user debug information are shown below.
_Code_
````
# Main Synthesis Script
puts "\nInfo: Creating main synthesis script to be used by Yosys"
set data "read_liberty -lib -ignore_miss_dir -setattr blackbox ${LateLibraryPath}"
set filename "$DesignName.ys"
set fileId [open $OutputDirectory/$filename "w"]
puts -nonewline $fileId $data

set netlist [glob -dir $NetlistDirectory *.v]
foreach f $netlist {
	set data $f
	puts -nonewline $fileId "\nread_verilog $f"
}
puts -nonewline $fileId "\nhierarchy -top $DesignName"
puts -nonewline $fileId "\nsynth -top $DesignName"
puts -nonewline $fileId "\nsplitnets -ports -format ___\ndfflibmap -liberty ${LateLibraryPath}\nopt"
puts -nonewline $fileId "\nabc -liberty ${LateLibraryPath}"
puts -nonewline $fileId "\nflatten"
puts -nonewline $fileId "\nclean -purge\niopadmap -outpad BUFX2 A:Y -bits\nopt\nclean"
puts -nonewline $fileId "\nwrite_verilog $OutputDirectory/$DesignName.synth.v"
close $fileId
puts "\nInfo: Synthesis script created and can be accessed from path $OutputDirectory/$DesignName.ys"

puts "\nInfo: Running synthesis................"
````

_Screenshots_

![Screenshot 2025-06-09 000852](https://github.com/user-attachments/assets/c6794f86-7425-4884-b9c3-6966bec46887)

_openMSP430.ys_

![Screenshot 2025-06-09 000927](https://github.com/user-attachments/assets/b886ba1f-b07d-4f71-8246-24c94607c980)

**Running main synthesis script & error handling**
I have successfully written the code for running the main Yosys synthesis script and exiting if errors are found. The basic code and screenshots of the terminal are shown below.
_Code_
````
if {[catch {exec yosys -s $OutputDirectory/$DesignName.ys >& $OutputDirectory/$DesignName.synthesis.log} msg]} {
	puts "\nError: Synthesis failed due to errors. Please refer to log $OutputDirectory/$DesignName.synthesis.log for errors"
	exit
} else {
	puts "\nInfo: Synthesis finished successfully"
}
puts "Please refer to log $OutputDirectory/$DesignName.synthesis.log"
````
_Screenshots_
Wantedly added an error to check whether the code is working or not 

![Screenshot 2025-06-09 001259](https://github.com/user-attachments/assets/d2f4b427-3500-4e2a-b0a6-39d4a3c6599c)

_openMSP430.synthesis.log_

![Screenshot 2025-06-09 001529](https://github.com/user-attachments/assets/be1aa69e-294f-48e3-98c0-e0f99232399d)

**Editing .synth.v to the format required for OpenTimer**
I have successfully written the code to edit the main synthesis output netlist .synth.v to make it usable for OpenTimer and other STA and PnR needs by replacing lines with "*" as a word and by removing "" from any and all lines that have it. The basic code of the same and screenshots of the terminal with several "puts" printing out the variables and user debug information are shown below.

_Code_
````
#Editing synth.v to be usable by Opentimer
set fileId [open /tmp/1 "w"]
puts -nonewline $fileId [exec grep -v -w "*" $OutputDirectory/$DesignName.synth.v]
close $fileId

set output [open $OutputDirectory/$DesignName.final.synth.v "w"]

set filename "/tmp/1"
set fid [open $filename r]
	while {[gets $fid line] != -1} {
		puts -nonewline $output [string map {"\\" ""} $line]
		puts -nonewline $output "\n"
	}
	close $fid
	close $output

	puts "\nInfo: Please find the synthesized netlist for $DesignName at below path. You can use this netlist for STA or PNR"
puts "\n$OutputDirectory/$DesignName.final.synth.v"
````
_Screenshots_

![Screenshot 2025-06-09 001753](https://github.com/user-attachments/assets/26b11663-c60a-4568-a518-44dd07bb90e2)

![Screenshot 2025-06-09 001812](https://github.com/user-attachments/assets/a2d7db67-1635-404e-bd2a-2025a8f4e545)

### World of procs
Procs can be used to create user-defined commands, as shown below. I have successfully written the code for all the procs. The basic codes of all the procs and screenshots of the terminal with several "puts" printing out the variables and user debug information for the 'set_multi_cpu_usage' and the 'read_sdc' procs are shown below.

**reopenStdout.proc**
This proc redirects the 'stdout' screen log to the file in the proc's argument.
_Code_
#!/bin/tclsh

# proc to redirect screen log to file
````
proc reopenStdout {file} {
    close stdout
    open $file w       
}
````
**set_multi_cpu_usage.proc**
This proc outputs multiple threads of the CPU usage command required for the OpenTimer tool.
_Code_
````
#!/bin/tclsh

proc set_multi_cpu_usage {args} {
        array set options {-localCpu <num_of_threads> -help "" }
        while {[llength $args]} {
                switch -glob -- [lindex $args 0] {
                	-localCpu {
				set args [lassign $args - options(-localCpu)]
				puts "set_num_threads $options(-localCpu)"
			}
                	-help {
				set args [lassign $args - options(-help) ]
				puts "Usage: set_multi_cpu_usage -localCpu <num_of_threads> -help"
				puts "\t-localCpu - To limit CPU threads used"
				puts "\t-help - To print usage"
                      	}
                }
        }
}
````
_Usage_

![image](https://github.com/user-attachments/assets/90750cee-a654-457c-8116-2a4b56b68e0a)

**read_lib.proc**
This proc outputs commands to read early and late libraries required for the OpenTimer tool.
_Code_
````
#!/bin/tclsh

proc read_lib args {
	# Setting command parameter options and its values
	array set options {-late <late_lib_path> -early <early_lib_path> -help ""}
	while {[llength $args]} {
		switch -glob -- [lindex $args 0] {
		-late {
			set args [lassign $args - options(-late) ]
			puts "set_late_celllib_fpath $options(-late)"
		      }
		-early {
			set args [lassign $args - options(-early) ]
			puts "set_early_celllib_fpath $options(-early)"
		       }
		-help {
			set args [lassign $args - options(-help) ]
			puts "Usage: read_lib -late <late_lib_path> -early <early_lib_path>"
			puts "-late <provide late library path>"
			puts "-early <provide early library path>"
			puts "-help - Provides user deatails on how to use the command"
		      }	
		default break
		}
	}
}
````

**read_verilog.proc**
This proc outputs commands to read the synthesised netlist required for the OpenTimer tool.
_Code_
````
#!/bin/tclsh

# Proc to convert read_verilog to OpenTimer format
proc read_verilog {arg1} {
	puts "set_verilog_fpath $arg1"
}
````
**read_sdc.proc**
This proc outputs commands to read constraints .timing file required for the OpenTimer tool. This procs converts SDC file contents to .timing file format for use by the OpenTimer tool, and the conversion code is explained stage by stage with sufficient screenshots.
Converting 'create_clock' constraints
Initially, the proc takes the SDC file as an input argument or parameter and processes the 'create_clock' constraints part of SDC.
_Code_
````
#!/bin/tclsh

proc read_sdc {arg1} {

# 'file dirname <>' to get directory path only from full path
set sdc_dirname [file dirname $arg1]
# 'file tail <>' to get last element
set sdc_filename [lindex [split [file tail $arg1] .] 0 ]
set sdc [open $arg1 r]
set tmp_file [open /tmp/1 w]

# Removing "[" & "]" from SDC for further processing the data with 'lindex'
# 'read <>' to read entire file
puts -nonewline $tmp_file [string map {"\[" "" "\]" " "} [read $sdc]]     
close $tmp_file

# Opening tmp file to write constraints converted from generated SDC
set timing_file [open /tmp/3 w]

# Converting create_clock constraints
# -----------------------------------
set tmp_file [open /tmp/1 r]
set lines [split [read $tmp_file] "\n"]
# 'lsearch -all -inline' to search list for pattern and retain elementas with pattern only
set find_clocks [lsearch -all -inline $lines "create_clock*"]
foreach elem $find_clocks {
	set clock_port_name [lindex $elem [expr {[lsearch $elem "get_ports"]+1}]]
	set clock_period [lindex $elem [expr {[lsearch $elem "-period"]+1}]]
	set duty_cycle [expr {100 - [expr {[lindex [lindex $elem [expr {[lsearch $elem "-waveform"]+1}]] 1]*100/$clock_period}]}]
	puts $timing_file "\nclock $clock_port_name $clock_period $duty_cycle"
}
close $tmp_file
````
**Code and Results for STA**
_Code_
````
puts "\nInfo: Timing Analysis Started....."
puts "\nInfo: Initializing number of threads, libraries, sdc, verilog netlist path...."
source /home/vsduser/vsdsynth/procs/reopenStdout.proc
source /home/vsduser/vsdsynth/procs/set_num_threads.proc
reopenStdout $OutputDirectory/$DesignName.conf
set_multi_cpu_usage -localCpu 2
#return

source /home/vsduser/vsdsynth/procs/read_lib.proc
read_lib -early /home/vsduser/vsdsynth/osu018_stdcells.lib

read_lib -late /home/vsduser/vsdsynth/osu018_stdcells.lib

source /home/vsduser/vsdsynth/procs/read_verilog.proc
read_verilog $OutputDirectory/$DesignName.final.synth.v

source /home/vsduser/vsdsynth/procs/read_sdc.proc
read_sdc $OutputDirectory/$DesignName.sdc
reopenStdout /dev/tty
#return
if {$enable_prelayout_timing == 1} {
	puts "\nInfo: enable_prelayout_timing is $enable_prelayout_timing. Enabling zero-wire load parasitics"
	set spef_file [open $OutputDirectory/$DesignName.spef w]
puts $spef_file "*SPEF \"IEEE 1481-1998\" "
puts $spef_file "*DESIGN \"$DesignName\" "
puts $spef_file "*DATE \"Sun May 11 20:51:50 2025\" "
puts $spef_file "*VENDOR \"PS 2025 Hackathon\" "
puts $spef_file "*PROGRAM \"Benchmark Parasitic Generator\" "
puts $spef_file "*VERSION \"0.0\" "
puts $spef_file "*DESIGN_FLOW \"NETLIST_TYPE_VERILOG\" "
puts $spef_file "*DIVIDER / "
puts $spef_file "*DELIMITER : "
puts $spef_file "*BUS_DELIMITER [ ] "
puts $spef_file "*T_UNIT 1 PS "
puts $spef_file "*C_UNIT 1 FF "
puts $spef_file "*R_UNIT 1 KOHM "
puts $spef_file "*L_UNIT 1 UH "
}

close $spef_file

set conf_file [open $OutputDirectory/$DesignName.conf a]
puts $conf_file "set_spef_fpath $OutputDirectory/$DesignName.spef"
puts $conf_file "init_timer "
puts $conf_file "report_timer "
puts $conf_file "report_wns "
puts $conf_file "report_worst_paths -numPaths 10000 "
close $conf_file

set tcl_precision 3

set time_elapsed_in_us [time {exec /home/vsduser/OpenTimer-1.0.5/bin/OpenTimer < $OutputDirectory/$DesignName.conf >& $OutputDirectory/$DesignName.results} 1]
set time_elapsed_in_sec "[expr {[lindex $time_elapsed_in_us 0]/100000}] sec"
puts "\nInfo: STA finished in $time_elapsed_in_sec seconds"
puts "\nInfo: Refer to $OutputDirectory/$DesignName.results for warning and errors"

puts "tcl_precision is $tcl_precision
````
_Screenshots_

![image](https://github.com/user-attachments/assets/f3251f6c-712a-4a60-a8c2-3c15dccce2e0)

![image](https://github.com/user-attachments/assets/02dce20c-dc5e-44c2-bfc7-f6939dc97b80)

![image](https://github.com/user-attachments/assets/34aece79-46d5-44eb-ae21-044d7340c02b)

![image](https://github.com/user-attachments/assets/fc6c7cb3-ca3f-4405-b38c-1a46e2c1c5f4)

**.spef generation**

![image](https://github.com/user-attachments/assets/88168765-0e3f-442b-a784-f5ead84e8fca)

![image](https://github.com/user-attachments/assets/1fab9e2d-5f57-436c-a36f-9112567d4c2d)

**.conf generation**

![image](https://github.com/user-attachments/assets/d154a5db-e2e4-4087-831c-b6ebc9d1a7bc)

![image](https://github.com/user-attachments/assets/b7ba4312-98c1-4905-a96b-2367d40827d3)


### Data collection from .results file and other resources for QoR
I have successfully written the code to collect all required data for specific codes and have also collected total .tcl script runtime. The basic code of the same and screenshots of the terminal with several "puts" printing out the variables and user debug information are shown below.

_Code_
````
#---------------find WNS using RAT-------------------------#
set worst_RAT_slack "-"
set report_file [open $OutputDirectory/$DesignName.results r]
set pattern {RAT}
puts "pattern is $pattern"
while {[gets $report_file line] != -1} {
	if {[regexp $pattern $line]} {
		set worst_RAT_slack "[expr {[lindex $line 3]/1000}]ns"
		puts "worst_RAT_slack is $worst_RAT_slack"
		break
	} else {
		continue
	}
}
close $report_file
#return

#--------------------------fine number of output violation------------#
set report_file [open $OutputDirectory/$DesignName.results r]
set count 0
while {[gets $report_file line] != -1} {
                incr count [regexp -all -- $pattern $line]
}
set Number_of_output_violations $count
puts "Number of output violations $Number_of_output_violations"
close $report_file


#---------------find WNS setup violation-------------------------#
set worst_negative_setup_slack "-"
set report_file [open $OutputDirectory/$DesignName.results r]
set pattern {Setup}
puts "pattern is $pattern"
while {[gets $report_file line] != -1} {
        if {[regexp $pattern $line]} {
                set worst_negative_setup_slack "[expr {[lindex $line 3]/1000}]ns"
                break
        } else {
                continue
        }
}
close $report_file

#---------------find number of setup violation-------------------------#

set report_file [open $OutputDirectory/$DesignName.results r]
set count 0
while {[gets $report_file line] != -1} {
                incr count [regexp -all -- $pattern $line]
}
set Number_of_setup_violations $count
close $report_file

#---------------find WNS hold violation-------------------------#
set worst_negative_hold_slack "-"
set report_file [open $OutputDirectory/$DesignName.results r]
set pattern {Hold}
puts "pattern is $pattern"
while {[gets $report_file line] != -1} {
        if {[regexp $pattern $line]} {
                set worst_negative_hold_slack "[expr {[lindex $line 3]/1000}]ns"
                break
        } else {
                continue
        }
}
close $report_file

#---------------find number of hold violation-------------------------#

set report_file [open $OutputDirectory/$DesignName.results r]
set count 0
while {[gets $report_file line] != -1} {
                incr count [regexp -all -- $pattern $line]
}
set Number_of_hold_violations $count
close $report_file


#---------------find number of instance---------------------------#
set pattern {Num of gates}
puts "pattern is $pattern"
set report_file [open $OutputDirectory/$DesignName.results r]
while {[gets $report_file line] != -1} {
        if {[regexp $pattern $line]} {
                set Instance_count [lindex [join $line " "] 4 ]
                break
        } else {
                continue
        }
}
close $report_file

set Instance_count "$Instance_count PS"
set time_elapsed_in_sec "$time_elapsed_in_sec PS"
````
_Screenshots_

![Screenshot 2025-06-09 234239](https://github.com/user-attachments/assets/b705bbaf-eaa2-4e1d-a5bf-2e6260426bed)

### QoR Genertion 
I have successfully written the code for QoR generation. The basic code and screenshots of the terminal are shown below.

_Code_
````
set formatStr {%15s%15s%15s%15s%15s%15s%15s%15s%15s}

puts [format $formatStr "-----------" "-------" "--------------" "-----------" "-----------" "----------" "----------" "-------" "-------"]
puts [format $formatStr "Design Name" "Runtime" "Instance Count" " WNS Setup " " FEP Setup " " WNS Hold " " FEP Hold " "WNS RAT" "FEP RAT"]
puts [format $formatStr "-----------" "-------" "--------------" "-----------" "-----------" "----------" "----------" "-------" "-------"]
foreach design_name $DesignName runtime $time_elapsed_in_sec instance_count $Instance_count wns_setup $worst_negative_setup_slack fep_setup $Number_of_setup_violations wns_hold $worst_negative_hold_slack fep_hold $Number_of_hold_violations wns_rat $worst_RAT_slack fep_rat $Number_of_output_violations {
	puts [format $formatStr $design_name $runtime $instance_count $wns_setup $fep_setup $wns_hold $fep_hold $wns_rat $fep_rat]
}

puts [format $formatStr "-----------" "-------" "--------------" "-----------" "-----------" "----------" "----------" "-------" "-------"]
puts "\n"
````
_Screenshots_

![Screenshot 2025-06-09 234513](https://github.com/user-attachments/assets/9c57e57d-33dd-4fe0-920c-e94df02870d6)


_Important screenshots of the course_

![Screenshot 2025-06-09 234350](https://github.com/user-attachments/assets/7e5ef49c-7978-476f-bd65-d37417b982c1)
![Screenshot 2025-06-09 234442](https://github.com/user-attachments/assets/1be93863-730a-40aa-9e50-17e87f79cd19)
![Screenshot 2025-06-09 234459](https://github.com/user-attachments/assets/0d2e83b0-8c06-4468-a9ba-5b9f8d6fc02c)
![Screenshot 2025-06-09 234537](https://github.com/user-attachments/assets/7ed2e96d-b32f-47b6-b22b-ba84dd67dd76)










