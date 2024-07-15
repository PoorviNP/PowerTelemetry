# PowerTelemetry
Creating a solution to monitor telemetry data (CPU, memory, NIC, and TDP) and simulate system load using containers can be achieved using several tools. Here is a step-by-step guide using Python, Docker, and some monitoring tools.

Step 1: Install Required Tools
You need the following tools installed on your system:

Docker
Python
psutil Python library for monitoring system performance
docker Python library to interact with Docker
Step 2: Create Python Script for Telemetry Data
Create a Python script to monitor the telemetry data

python

import psutil

import time

import docker

def get_telemetry():

     telemetry = {

            'cpu': psutil.cpu_percent(interval=1),
    
            'memory': psutil.virtual_memory().percent,
    
            'nic': psutil.net_io_counters().bytes_sent + psutil.net_io_counters().bytes_recv,
   
            'tdp': psutil.sensors_temperatures().get('coretemp', [])[0].current if 'coretemp' in psutil.sensors_temperatures() else 'N/A'

             }

       return telemetry

def run_container_load(utilization):

      client = docker.from_env()

      container = client.containers.run("stress", f"--cpu 1 --io 1 --vm 1 --vm-bytes {utilization}% -t 60s", detach=True)

      return container

def monitor_system(utilization):

     container = run_container_load(utilization)

     while container.status != 'exited':

         telemetry = get_telemetry()
    
         print(f"Telemetry Data: {telemetry}")
    
     time.sleep(5)  # Adjust the sleep time as per requirement
    container.reload()

if __name__ == "__main__":

     utilization = int(input("Enter the percentage of system utilization: "))

     monitor_system(utilization)
Step 3: Create Dockerfile for Load Generation
Create a Dockerfile to build an image that generates load. We'll use the stress tool for this.

Dockerfile

FROM alpine:latest

RUN apk add --no-cache stress

ENTRYPOINT ["stress"]
Build the Docker image:

sh

docker build -t stress .
Step 4: Running the Solution
Ensure Docker is running.

Run the Python script.

sh

python monitor_system.py
Explanation
Telemetry Data Collection:

psutil library is used to gather CPU, memory, NIC, and temperature data (TDP).
get_telemetry function returns a dictionary of these metrics.
Run Container Load:

Docker Python library is used to run a container that uses the stress tool to generate load.
The run_container_load function starts the container with the specified CPU and memory utilization.
Monitor System:

The monitor_system function continuously monitors the system telemetry while the container is running.
The script prints telemetry data every 5 seconds until the load container exits.
