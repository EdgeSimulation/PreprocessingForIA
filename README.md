# OMNeT++ and SUMO datasets adaptation for proactive edge computing approaches

This repository includes some Jupyter Notebooks and commands used to preprocess custom datasets generated from simulation to be ready for training and prediction of user device trajectories in edge computing scenarios. Some tools and steps to extract significant information and aggregate the necessary data from OMNeT++ and SUMO generated datasets are listed and described below.

OMNeT++ and SUMO datasets are obtained though simulation based on the workflow indicated in paper "End-to-end simulation environment for mobile edge computing", by using the instructions and code provided in the repository https://github.com/EdgeSimulation/EdgeSimulation.

In order to obtain a combined dataset with data from OMNeT++ and SUMO, it is essential to guarantee the equivalence of vehicle ids in SUMO and the linked objects in OMNeT++ as, by default, there might be some differences that lead to problems when combining both datasets. 

## STEP 0: Ensuring SUMO and OMNeT++ vehicles equivalence

Vehicle ids need to be extracted from SUMO and from Veins in OMNeT++ as a scalar statistic for each vehicle modifying VeinsInetMobility in Veins. Concretely, we have created a new scalar statistic named ```carNumSUMO```, and we have also added a couple of lines in the file named ```veins/modules/application/traci/TraCIDemo11p.cc``` introducing a code similar to these lines that permit the ids to be saved on the scalar output file ```scalar-0.sca```.

       EV << "My SUMO id = " << getExternalId() << " - My VEINS id = " << getParentModule()->getIndex() << endl;

Once obtained OMNeT++ simulation results, in order to extract vehicle ids from OMNeT++, the scalar output file has to be filtered to get the corresponding id of vehicles in Veins and SUMO. We have used the following command to get the equivalence:

       $ grep -A2 'carNumSUMO:stats' scalar-0.sca | grep '[0-9]\+' -o | paste -d " "  - - - | awk '{print $1" "$3}' > carxxxx.txt

Being ```xxxx``` the id of the simulation (that in our case refers to the total number of vehicles considered in the process).

# Steps to process a SUMO dataset for AI

These steps do not include the necessary configuration of the urban coverage area and routes generation but provide the instructions to extract all signals that can be obtained from SUMO configuration files (```Alicante_4928.sumo.cfg``` in our case) and routes files (i.e. ```Alicante_4928_routes-rou.xml```, ```Alicante_centro_ciudad.net.xml```). 

## STEP 1: Obtain all signals from SUMO

In order to obtain all possible signals (i.e. ```fdc_signals_xxxx.xml```) of vehicles from SUMO trajectories this instruction can be used on the command line:

       $ sumo -c Alicante_4928.sumo.cfg --fcd-output.geo true --fcd-output.signals true --fcd-output ../fdc_signals_4928.xml --end 1800

## STEP 2: Extract data from SUMO

Run ```dataset_AI_sumoID.ipynb``` notebook to obtain formatted files with all values extracted from SUMO. 
Check the mentioned notebook as it has three functions defined, one to organise the information in table format, one to add the geographical coordinates of the 7 past seconds (```x-1,y-1 .. x-7,y-7```) and another to get the future 7 future second geo positioning  (```x1,y1 .. x7,y7```) of each vehicle, that might be useful depending on the AI algorithm planned to be used.  

# Steps to process an OMNeT++ dataset for AI

Considering that the OMNeT++ generated files per simulation are named  (```vector-0.vec```, ```scalar-0.sca```, ```vector-0.vci```), we proceed to adapt the vector dataset for AI following the next steps.

## STEP 3: Get attributes from OMNET++ output vectors

Run ```dataset_AI_omnet.ipynb``` notebook to obtain the desired attributes from OMNeT++ vector file  ```vector-0.vec```. In our case, we are interested in obtaining values of  ```distance```, ```measuredSinrDl```, ```measuredSinrUl```, ```rcvdSinrDl```, ```averageCqiDl```, ```servingCell```, ```rlcDelayDl```, ```rlcPacketLossTotal, ``` ```rlcPduDelayDl```, ```rlcPduPacketLossDl```, ```rlcPduThroughputDl```, ```rlcThroughputDl```, ```receivedPacketFromLowerLayer```.

## STEP 4: Format OMNeT++ dataset as a matrix

Once obtained the desired attributes, files ```xxxx_omnet_AI.csv``` have to be formatted as matrices named ```xxxx_omnet_AI_matrix.csv``` with notebook  ```OMNET.ipynb```.

# Steps to combine OMNeT++ and SUMO datasets

In order to combine SUMO and OMNeT++ datasets we have to ensure the ids of the vehicles in SUMO and the linked objects in OMNeT++ are equivalent as, by default there might be some differences that lead to problems when combining both datasets. 

## STEP 5: Combine SUMO and OMNeT++ datasets

Notebook named ```merge_files_sumo_omnet_xxxx.ipynb``` combines SUMO and OMNeT++ datasets based on vehicle ids equivalence.  Entry files are  ```xxxx_sumo_AI.csv```, ```xxxx_omnet_AI_matrix.csv``` and ```carxxxx.txt ```.
