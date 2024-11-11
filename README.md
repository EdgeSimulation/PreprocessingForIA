# OMNeT++ and SUMO datasets adaptation for proactive edge computing approaches

This repository includes some Jupyter Notebooks and commands used to preprocess custom datasets generated from simulation that are ready to be trained and predict user devices trajectories in edge computing scenarios. Some tools and steps to extract significant information and aggregate the necessary data from OMNeT++ and SUMO generated datasets are listed and described below.

OMNeT++ and SUMO datasets are obtained though simulation based on the workflow indicated in  paper "End-to-end simulation environment for mobile edge computing", by using the instructions and code provided in repository ```https://github.com/EdgeSimulation/EdgeSimulation```.

# Steps to process a SUMO dataset for AI

These steps do not include the necessary configuration of the urban coverage area and routes generation but provide the instructions to extract all signals that can be obtain from SUMO configuration file (```Alicante_8620.sumo.cfg``` in our case), routes files (i.e. ```Alicante_8620_routes-rou.xml```, ```Alicante_centro_ciudad.net.xml```) 

## STEP 1: Obtain all signals from SUMO

In order to obtain all possible signals (```fdc_signals_xxxx.xml```) of vehicles from SUMO trajectories type this instruction can be used on the command line:

```$ sumo -c Alicante_8620.sumo.cfg --fcd-output.geo true --fcd-output.signals true --fcd-output ../fdc_signals_8620.xml --end 1800```

## STEP 2: Extract data from SUMO

Run ```dataset_AI_sumoID.ipynb``` notebook to obtain formatted files with all values extracted from SUMO. 
Check the mentioned notebook as, apart from organising the information, there are defined two functions to obtain the coordinates of either the 7 past seconds (```x-1,y-1 .. x-7,y-7```) or future seconds (```x1,y1 .. x7,y7```) of each vehicle, that might be useful depending on the AI algorithm planned to be used. 

# Steps to process an OMNeT++ dataset for AI

Considering OMNeT++ generated files per simulation named  (```vector-0.vec```, ```scalar-0.sca```, ```vector-0.vci```), we proceed to adapt the dataset following the next steps.

## STEP 1: Get attributes from OMNET output vectors

Run ```dataset_AI_omnet.ipynb``` notebook to obtain the desired attributes from OMNeT+ vector file  ```vector-0.vec```. In our case, we are interested in values of  ```distance```, ```measuredSinrDl```, ```measuredSinrUl```, ```rcvdSinrDl```, ```averageCqiDl```, ```servingCell```, ```rlcDelayDl```, ```rlcPacketLossTotal, ``` ```rlcPduDelayDl```, ```rlcPduPacketLossDl```, ```rlcPduThroughputDl```, ```rlcThroughputDl```, ```receivedPacketFromLowerLayer```.

## STEP 2: Format OMNeT++ dataset as a matrix

Once obtained the desired attributes, files ```xxxx_omnet_AI.csv``` have to be formatted as matrices named ```xxxx_omnet_AI_matrix.csv``` with notebook  ```OMNET.ipynb```, being ```xxxx``` the id of the simulation (that in our case refers to the total number of cars considered in the process).

# Steps to combine OMNeT++ and SUMO datasets

In order to combine SUMO and OMNeT++ datasets we have to ensure the ids of the cars in SUMO and the linked objects in OMNeT++ are equivalent as, by default there might be some differences that lead to problems when combining both datasets. 

## STEP 1: Ensuring SUMO and OMNeT++ cars equivalence

Car ids need to be extracted from SUMO and from VEINS in OMNeT++ as a scalar statistic for each car. A possible implementation is modifying a VEINS file named ```veins/modules/application/traci/TraCIDemo11p.cc``` by introducing a code similar to these lines (ids are saved on file ```scalar-0.sca```):

   ```EV << "My SUMO id = " << mobility->getExternalId() << endl;```
   ```EV << "My VEINS id = " << getParentModule()→getIndex() << endl;```

## STEP 2: Extract car ids from OMNeT output file

OMNeT++ scalar output file has to be filtered to obtain the corresponding id of vehicles in Veins and SUMO. We have used the following command to get the equivalence:

```$ grep -A2 'carNumSUMO:stats' scalar-0.sca | grep '[0-9]\+' -o | paste -d " "  - - - | awk '{print $1" "$3}' > carxxxx.txt ```

## STEP 3: Combine SUMO and OMNeT++ datasets

Notebook named  ```merge_files_sumo_omnet_xxxx.ipynb``` combines SUMO and OMNeT++ datasets based on vehicle ids equivalence.  Entry files are  ```xxxx_sumo_AI.csv```, ```xxxx_omnet_AI_matrix.csv``` and ```carxxxx.txt ```
