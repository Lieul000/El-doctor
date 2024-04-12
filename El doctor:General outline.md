# El Doctor/General outline

Creating a custom monitoring script can be an exciting and rewarding project. Here's a general outline to get you started on developing your own monitoring tool:

- Choose Your Programming Language: Select a programming language that you're comfortable with and that is suitable for system monitoring tasks. 
Bash will be my choice due to straightforwardness and quicker implementation options.

- Define Metrics to Monitor: Decide on the metrics you want to monitor. 
I will go with  CPU usage, memory usage, disk space.

- Design the Script Structure: Plan how your script will be organized. 
This is how I break it down - collecting data, storing data in cvs, running with cron, notifications by mail and getting a weekly report.

Implement Data Collection: Write code to collect the chosen metrics. 

Store Data in CSV File: Develop functionality to store the collected metrics in a CSV file. 

Implement Alerts: Set up alerts to notify you via email or other means if certain thresholds are exceeded (e.g., low disk space, high CPU usage). 

Automate Execution: Use cron or a similar scheduling tool to automate the execution of your monitoring script at regular intervals. 

Generate Weekly Reports: Develop a feature to generate weekly reports summarizing the collected metrics. 

- Testing and Refinement: Test your monitoring script thoroughly to ensure it behaves as expected. Make any necessary refinements or optimizations based on testing results and feedback.

