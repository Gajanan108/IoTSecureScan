 IoTSecureScan
The tool demonstrates how automated scanning and vulnerability identification can be integrated into IoT networks
 IoT SAST Tool: Firmware Vulnerability Scanner

 Overview

This tool is designed to perform Static Application Security Testing (SAST) on IoT firmware to identify common vulnerabilities as per the OWASP IoT Top 10. The tool integrates with SonarQube for vulnerability scanning and generates comprehensive security reports. The tool primarily focuses on detecting issues such as hardcoded passwords, insecure services, lack of secure update mechanisms, and more, using SonarQubes static analysis capabilities.

 Features

- SAST Analysis for IoT Firmware: Analyze IoT firmware to detect security vulnerabilities.
- SonarQube Integration: Uses SonarQubes API to identify vulnerabilities based on CWE identifiers.
- CSV Report Generation: Outputs detailed reports in CSV format containing information on vulnerabilities, severity, and CWE identifiers.
- Customizable Reports: Generates CSV files with vulnerability details including `Component`, `Issue`, `Severity`, `Location`, `CWE ID`, and `Description`.

 Requirements

- Python 3.x: The script is written in Python 3.x.
- SonarQube: A SonarQube instance is required to run the analysis. You can set up SonarQube locally or use a cloud service.
- Internet Connection: For fetching vulnerabilities and sending data to SonarQube.

 Dependencies

The following Python libraries are required:

- `requests` – To interact with SonarQube’s API.
- `csv` – For handling CSV files.
- `sonar-scanner` – SonarQube scanner CLI to run analysis (if integrating locally).

You can install the necessary Python packages via pip:

```bash
pip install requests


