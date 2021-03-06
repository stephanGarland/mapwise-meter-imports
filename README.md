# mapwise-meter-imports

All three of these macros are intended to be used with NISC Mapwise and MDMS software. It's highly unlikely that it will be applicable to you otherwise, although you can probably use some of the column sorting to clean up nasty CSV files.

It's not the prettiest code. It works.

HOWTO: Import a Mapwise Layer Attribute Export

This describes how to take a CSV file exported from Mapwise containing layer attributes such as gs\_account\_number, gs\_rate\_schedule, etc. and format it into one that makes more sense, using a VBA macro in Excel. The macro will set up the workbook for further analysis with kWh usage from MDMS, and generate a meter list CSV file to use with creating a Virtual Electric Meter (VEM) in MDMS [optional].

_Usage_

1. Open the CSV file generated by Mapwise, and navigate to the Developer Tab.
2. Click on Visual Basic, and if there is code already present, delete all of it.
3. Open the &quot;VBA Mapwise Data Import.txt&quot; file, and copy/paste all of it into the VB window. **CAUTION: ENSURE WORD WRAP (FORMAT**  **-->** **WORD WRAP IF USING NOTEPAD) IS DISABLED.**
4. Scroll up to the top of the code, and click to place the cursor in the first subroutine (Cleanup\_Mapwise\_Import).
5. Click the green triangle, or press F5 to run the macro.
6. You will be asked if you want to delete the original sheet, keeping only the new one. This is entirely up to you, as you can simply save the new sorted sheet as a new file. If you choose Yes, Excel will ask to confirm that you want to delete the sheet.
7. You will next be asked if you want to create a separate CSV file that contains only meter numbers for creating a Static Meter List to import into MDMS. **NOTE: WHILE YOU CAN ALSO DO THIS STEP MANUALLY, THE SUBROUTINE ALSO SEARCHES FOR AND REMOVES BLANKS AND INVALID METERS NUMBERS (0), BOTH OF WHICH CAN CAUSE ERRORS IN MDMS.**
8. Depending on which options you selected, you&#39;ll now have one or two workbooks, with the original containing three sheets – Data, Contributors, and Graph. Data will have three columns – Date/Time, kW, and Amps. The cleaned data imported from Mapwise is on Contributors, and Graph is empty. These sheets are used in another macro, VBA Meter Custom Report. If all you need is the information about the accounts, you can simply stop here and delete the extraneous sheets.

_Modification_

If you need different data fields, open the macro, and change the fields in the first two arrays – targetCols, and replColNames. targetCols must contain the fields exactly as written in the original CSV, and replColNames must be in the same order, with your desired name. **CAUTION: THE MACRO DOES NO SANITY CHECKING, AND WILL HAPPILY PUT GS\_METER\_NUMBER INTO A FIELD NAMED CITY IF YOU DO NOT PAY ATTENTION.**

_Design Notes_

I am not a VBA expert. The code isn&#39;t as great as it could be, and isn&#39;t very resilient to change. Mea culpa. Specifically, the int counter is used as both 0 and 1-indexed, and if you change this without fixing headerIndex, the macro breaks in interesting ways.

----------------------------------------------------------------------------------------------------------------------------------------

HOWTO: Custom Meter List Analysis

This describes how to take the resulting Excel workbook from VBA Mapwise Data Import, create a Virtual Electric Meter (VEM) in MDMS, and use the VBA Meter Custom Report macro to present the data in a useful way.

_Preparation_

1. Open MDMS, and navigate to Management Assets. **NOTE: IF THIS ISN&#39;T AVAILABLE, CONTACT YOUR MDMS ADMIN TO GAIN RIGHTS, OR HAVE THEM CREATE THE METER.**
2. Click Create Virtual Electric Meter.
3. Type a descriptive name into Asset ID. All other fields are optional. Click Save.
4. Once saved, go to the Contributing Assets tab, and select Static List.
5. Click Import Meters, and import the CSV file containing only the meter numbers. **NOTE: IF YOU RECEIVE AN ERROR ABOUT UNBOUND RANGES, IT IS HIGHLY LIKELY THAT THE METERS ARE FORMATTED INCORRECTLY. CHECK FOR LEADING/TRAILING WHITESPACE, NULLS, ETC.**
6. Go to the Channels tab, and click Add Channel.
7. Insert: Channel Number: 1, Unit of Measure: kWh, Flow Direction: Forward.
8. Go to the Input Mappings sub-tab, and type a descriptive name. **NOTE: NO WHITESPACE IS ALLOWED HERE, RECOMMEND UNDERSCORES.**
9. Insert: Adjustment Factor: 1, Unit: kWh, Channel: 1, Flow: Forward.
10. Click Save Input Mapping, then Save Channel.
11. Go to the Aggregation tab, and insert your desired date range. **NOTE: THERE IS NO NEED TO HAVE PUT IN A DATE RANGE IN ANY PREVIOUS STEPS.**
12. Once your date range is in, click Start. Depending on the range and number of meters, this could take several minutes.
13. Once complete, you can go to Metering Measurements, and download the meter as usual. **CAUTION: USE HOURLY READINGS ONLY.**

_Usage_

1. Open the Excel workbook generated by VBA Mapwise Data Import, and the CSV file of usage downloaded from MDMS.
2. Copy Start Date Time and Value columns into the Data sheet of your Excel workbook. **WARNING: THE MACRO ASSUMES YOU ARE COPYING JANUARY – DECEMBER. IF YOU HAVE A LARGER RANGE THAN THAT, YOU WILL NEED TO MAKE MODIFICATIONS TO THE MACRO. IF YOU SIMPLY HAVE A DIFFERENT 12-MONTH WINDOW, CHANGE THE TABLE MONTHS AFTER THEY ARE GENERATED. NOTE: DO NOT COPY THE HEADERS, ONLY THE DATA.**
3. In your Excel workbook, navigate to the Developer Tab. You can close the MDMS-generated file.
4. Click on Visual Basic. In the left pane, find your Workbook, and select &quot;This Workbook.&quot; **CAUTION: IF YOU DO NOT CHANGE FROM THE OPENING SELECTION (RIBBON), YOUR DATA TAB WILL MOST LIKELY THROW ERRORS UNTIL YOU CLOSE AND RESTART EXCEL. NOTE: IF YOU SEE &quot;OPTION EXPLICIT&quot; AT THE TOP OF THE RIGHT PANE, DELETE IT, OR THE ONE INCLUDED IN THE SCRIPT – IT CAN ONLY APPEAR ONCE.**
5. Open the &quot;VBA Meter Custom Report.txt&quot; file, and copy/paste all of it into the VB window. **CAUTION: ENSURE WORD WRAP (FORMAT**  **-->** **WORD WRAP IF USING NOTEPAD) IS DISABLED.**
6. Scroll up to the top of the pasted code, and place the cursor into the RunMeFirst() subroutine.
7. Click the green triangle, or press F5 to run the macro. **NOTE: IF YOU PLACE THE CURSOR ABOVE OR BELOW THIS, EXCEL WILL SIMPLY ASK YOU WHAT SUBROUTINE YOU WANT TO RUN. YOU CAN DIRECTLY CHOOSE COPYTABLES, OR SELECT RUNMEFIRST.**
8. If you opted to have the macro create an ACSR data table for you, select the codeword for your primary conductor, e.g. Sparate on the Data worksheet.

_Modification_

1. If you understand VBA, I assume you can modify the macro. It&#39;s documented fairly well.

_What The Macro Accomplishes_

1. It creates four tables on the Data worksheet – Hourly Maximums (Primary kW), Hourly Maximums (Primary Amps), an adjustment table containing Primary Voltage and Power Factor, and a Phase Count.
2. The Phase Count relies on the accuracy of the imported data from Mapwise, and simply counts up the number of members with each listed phase. If your three-phase isn&#39;t labeled as ABC, you&#39;ll need to change cell Q12 (currently listed as ABC under Phase in the table) to that value.
3. The two Hourly Maximums tables have Conditional Formatting applied – kW uses blue gradient bars, and Amps uses either an arbitrary color scale, or if you opted to have the ACSR table created, it will allow you to select a conductor to see loading as a percentage of its ampacity.
4. Things are made visually appealing. This is subjective, but if you disagree, you&#39;re wrong.
5. It creates an auto-filled Amps column which is derived from the reported kW. This relies on the fact that a kWh meter reporting hourly is in fact reporting kW, and then uses the following equation: A=kW∗1000V∗PF **NOTE 1: THIS CALCULATION ASSUMES THAT THE KW IS AN AGGREGATE OF SINGLE-PHASE METERS ULTIMATELY TERMINATING IN THREE-PHASE. NOTE 2: VPRI IS PHASE-NEUTRAL. DO NOT USE PHASE-PHASE VOLTAGES.**
6. Expanding on Note 1: To calculate three-phase amps, you divide the single-phase result by three, assuming perfect balance between phases. As the majority of members are not three-phase, this attempts to be reasonably accurate by aggregating them together, and then later dividing out the total load by the phase count. This is more accurate with a homogenous load base, such as residential areas. Feeders with large numbers of phase converters or three-phase accounts will skew the numbers. Take them with a grain of salt, and check it against your engineering model.
7. It creates a Clustered Column graph on the sheet of the same name, containing kW vs. Time. Feel free to alter the graph as you see fit.
----------------------------------------------------------------------------------------------------------------------------------------
HOWTO: Merged Line Analysis

This describes how to take the resulting Excel workbook from two VBA Mapwise Data Import, and merge them to simulate line loading. It is not a replacement for an engineering model, but a decent quick-and-dirty.

_Preparation_

1. Use the other macros and MDMS to create two individual .xlsx files containing the meters you wish to model. These are most likely created from downstream/upstream traces in Mapwise, modeling shutting an air brake or the like.

_Usage_

1. Open a new spreadsheet, and following the same precautions as in the single meter analysis, import and run this macro.
2. Navigate to and select your first .xlsx file to be created, and then the second.
3. While you are given the option to not create an ACSR worksheet, it is highly recommended that you do so.

_Modification_

1. Same as before; if you can, then do so. There are a couple of gnarly R1C1 formulas in there, so have fun with that.

_What The Macro Accomplishes_

1. It basically does the same thing as the Custom Meter Analysis, but x2.
2. It creates a named sheet for each Contributors sheet from the sources, and then makes a combined column on Data to add the two up.
3. It does NOT create a graph, as I felt it was redundant. Feel free to insert your own.

   *Stephan Garland*
   
   *stephan.marc.garland@gmail.com*
