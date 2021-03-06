# Architecture

## The Ercole Project

![Architecture](/architecture.png "Architecture")

The Ercole projects is made up of the following components/projects:

- __Ercole Server:__ Provides the data collection endpoint and methods for the User Interface.
- __Ercole Web:__ the user interface: dashboard, alerts, host details, etc.
- __Ercole Agent(s):__ Data collectors to be installed on the target machines.

Ercole Agents are typically packaged as daemons / services for the supported platforms.
Once started, the agent periodically executes a set of operations:

- launch a series of 'fetchers': shell commands to retrieve System and Oracle instance information
- transform the collected data into a JSON object
- send the data to the Ercole Server via a HTTP POST request

The Ercole Server is responsible of:

- Receiving the host data from the agent via HTTP
- Storing the data in a PostgreSQL database
- Archiving previous host information if already present in the db
- Processing the incoming host data and generating alerts (i.e., a new feature has been activated)
- Serving data to the Web User interface via a REST API

## Data format

The PostgreSQL database main tables are:

- __current_host:__ contains a few 'search-friendly' columns (hostname, databases, environment) and two columns (host_info and extra_info) to store the detailed json data for the host. Note the columns are LONGTEXT, casting
to json/jsonb happens at query level in the server methods.
- __historical_host:__ a copy of current_host, holds historical data
- __alert:__ alerts generated by the server
- __license:__ number of purchased Oracle Licenses (declared by the user in Ercole Web).

## Deployment

Currently Ercole-Server embeds Ercole agents and Ercole Web, so it can be deployed as a
standalone jar.

It is recommended to set up a proxy server (nginx, apache) in front of Ercole Server to provide
SSL communication between the agents and the server.

## Future versions

In the future (version 2.x) we're considering a few enhancements that will likely split up
the architecture and impact the data format:

![ArchitectureV2](/architecture-v2.png "ArchitectureV2")

- Move to mongodb or other json-based storage
- Create separate services:
  - Alert: Generate alerts, send notifications. Expose data for 3rd party usage (i.e. prometheus)
  - API: provides REST APIs for the User Interface
  - Data: receives data from the agent
  - Repo: provides a yum repository (proxy?) for the agent binaries
- Implement a self-update mechanism for the agents, move agent packages to a dedicated repository/service
- Use Go as preferred language for services

Note that this list is subject to change, new ideas, etc.
