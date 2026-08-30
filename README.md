# 5G Core Network Service-Based Architecture (SBA) Latency Analysis

This repository contains a comprehensive technical analysis of the 5G Service-Based Architecture (SBA). Developed as part of the Wipro 5G Datacom Project, this project isolates, correlates, and measures the microservice latency incurred during the User Equipment (UE) authentication phase across the Access and Mobility Management Function (AMF), Authentication Server Function (AUSF), and Unified Data Management (UDM).

## Project Overview

In legacy telecommunications architectures, core network functions communicated via rigid, point-to-point signaling protocols. The 5G Core introduces the Service-Based Architecture, heavily leveraging IT-standard protocols—specifically HTTP/2 over TCP—for Control Plane communication. 

While this cloud-native paradigm enables dynamic scaling and microservice orchestration, it requires multiple Network Functions (NFs) to communicate over network interfaces to complete a single user procedure. During a UE Initial Registration, the AMF cannot independently authenticate the subscriber. It must request authentication via the AUSF (`Nausf`), which in turn requests cryptographic vectors from the UDM (`Nudm`). 

This project establishes a live 5G Core environment, captures the resulting HTTP/2 API traffic during a registration event, and utilizes programmatic data analysis to quantify the exact processing delay introduced by the cryptographic algorithms and database lookups within these core microservices.

## Technical Stack and Architecture

The implementation and analysis environment relies on the following open-source frameworks and diagnostic tools:

* **Open5GS (Core Network):** An open-source, C-language implementation of the 5G Core, compiled from source to provide the AMF, AUSF, UDM, and UDR microservices.
* **UERANSIM (Radio Access Network):** A state-of-the-art 5G SA (Standalone) UE and gNodeB simulator used to generate 3GPP-compliant signaling and trigger the Initial Registration sequence.
* **Wireshark / TShark:** Utilized for deep packet inspection. TShark was specifically configured to dissect TCP port 7777 traffic as HTTP/2 and extract relevant JSON header payloads.
* **Python 3 Data Stack:** The `pandas` and `matplotlib` libraries were employed to ingest raw CSV packet data, correlate HTTP/2 stream requests with their respective responses, calculate microsecond latency differentials, and generate visual sequence diagrams.

## Repository Structure

The repository is structured to separate source code, raw data, and documentation for clear peer review and replication.

* `src/` 
  * `analyze_sbi_latency.py` : The Python script responsible for parsing the TShark CSV output, mapping IP addresses to NFs, calculating latency, and rendering the graphical breakdown.
* `data/` 
  * `sbi_registration_trace.pcap` : The raw, unfiltered packet capture containing the full HTTP/2 transmission.
  * `sbi_auth_metrics.csv` : The sanitized dataset extracted via TShark, containing only epoch timestamps, HTTP statuses, and URI paths.
* `assets/` 
  * `latency_breakdown_graph.png` : The generated bar chart visualizing the processing time required by the AUSF and UDM.
  * `execution_terminal_trace.png` : Verification of the script execution and textual sequence diagram output.
* `docs/` 
  * `Engineering_Analysis_Report.pdf` : The report detailing the methodology, architecture context, and strategic takeaways of the analysis.

## Execution and Reproduction Steps

For those wishing to reproduce this analysis in their own local environments, follow the steps below.

**1. Data Extraction via TShark**
Assuming the Open5GS Core and UERANSIM have been successfully deployed and a packet capture (`.pcap`) has been recorded during UE attachment, execute the following command to dissect the HTTP/2 headers and extract the necessary fields:

```bash
tshark -r data/sbi_registration_trace.pcap -d tcp.port==7777,http2 -Y "http2.header.path contains \"/nausf\" || http2.header.path contains \"/nudm\"" -T fields -e frame.time_epoch -e ip.src -e ip.dst -e http2.header.method -e http2.header.status -e http2.header.path -E separator=, > data/sbi_auth_metrics.csv
```

**2. Latency Computation and Visualization**

Ensure the Python environment is configured with the necessary data science libraries. Execute the analysis script to parse the CSV, correlate the API streams, and output the metrics.

```bash
pip install pandas matplotlib
python3 src/analyze_sbi_latency.py
```

**3. Key Findings and Metrics**

The computational analysis mapped the full authentication sequence and verified 3GPP compliance. The latency breakdown indicated that the UDM Processing Phase consumed the absolute majority of the transaction time.

This delay is an expected architectural behavior, as the UDM is tasked with retrieving the subscriber's permanent cryptographic key from the UDR (MongoDB database) and running the CPU-intensive Milenage/TUAK cryptographic algorithms to generate the Authentication Vectors (RAND, AUTN, HXRES*). By contrast, the AUSF overhead was negligible, acting primarily as a validation and routing layer between the AMF and UDM.

For exact millisecond metrics and further operational context, refer to the `Engineering_Analysis_Report.pdf` located in the `/docs` directory.


## License and Disclaimer

This project is submitted as an educational and analytical exercise. It relies on open-source software (Open5GS, UERANSIM) which are governed by their respective licenses (AGPL-3.0 and GPL-3.0). The configuration files, scripts, and analytical documentation provided in this repository are for research, evaluation, and portfolio demonstration purposes.


## Done by

* **Name**: Dinesh S
* **College Name**: Rajalakshmi Institute of Technology, Chennai
* **Year of Study and Department**: Final Year, B.E Electronics and Communication Engineering
* **College Registration Number**: 2117230040031
* **Contact**: dineshh.subramaniyan@gmail.com
