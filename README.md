# COVID_Simulation
The purpose of the project is to simulate the spread of COVID-19 in some specified countries, using a set of data that are provided. For this matter, I need to create a sample population of people according to the data provided (sample ratio is provided in the test.py, for example, one sample per one million population).
This should be done according to the age structure in each country (percentage of population in each age group).

Then for each individual in the sample, I have run a simulation (a Markov Chain similar to assignment 2) for a specific period of steps (days). Starting and ending dates are provided to you in (the test.py). The Markov chain simulation should consider 5 states (H, I, S, M, D) which are described later. This result needs to be exported to a CSV file (a3-covid-simulated-timeseries.csv).

At the end, I need to accumulate data for each specified country and each date (see the Output section). The result should be saved as (a3-covid-summary-timeseries.csv). Then I need to call the create_plot() function and to create a chart (a3-covid-simulation.png).
