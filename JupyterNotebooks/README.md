# Jupyter notebooks

The Jupyter notebooks are used in the different stages of the workflow as described in the upper layer README document.

This is a short description of the functionalities of each notebook:
* ```dataset_AI_sumoID``` - the notebook uses the SUMO log file once extracted the floating car data (FDC) as input and obtains most sumo parameters in a table format. It also creates datasets that adds the 7 previous geo positions or the 7 past geo positions of vehicles in each row. Resulting datasets are used as input in the SUMO and OMNeT++ merging step.
* ```dataset_AI_omnet``` - the notebook uses vector output files from  OMNeT++ simulation and extracts in files named as ```xxxx_omnet_AI.csv``` (being ```xxxx``` the number of vehicles considered in the simulation) a set of attributes from it ordered by time.
* ```generate_OMNeT_matrix_xxxx``` - the notebook filters OMNeT++ unnecessary data and provides a matrix shape file with important features.
* ```merge_files_sumo_omnet_xxxx``` -  this notebook is used to merge data obtained from Sumo and OMNeT ++ by indexing by vehicle’s id and time
