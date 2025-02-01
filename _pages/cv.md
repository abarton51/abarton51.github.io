---
layout: archive
title: "Resume"
permalink: /resume/
author_profile: true
redirect_from:
  - /resume
  - /cv
---

<!-- Trigger Button -->
<a href="#" id="open-modal">View or Download the PDF Version Here</a>

<!-- Modal -->
<div id="pdf-modal" style="display:none;">
  <div id="pdf-modal-content">
    <iframe src="{{ site.baseurl }}/assets/pdfs/Barton_Austin_T_Resume-2.pdf" width="100%" height="95%"></iframe>
    <button id="close-modal">Close</button>
  </div>
</div>

<!-- Add some basic styles for the modal -->
<style>
  #pdf-modal {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: rgba(0, 0, 0, 0.7);
    display: none;
    justify-content: center;
    align-items: center;
  }

  #pdf-modal-content {
    background: #11111a;
    padding: 20px;
    border-radius: 10px;
    width: 100%;
    max-width: 1000px;
    height: 80%;
    max-height: 90%;
    overflow: hidden;
    box-shadow: 0 4px 10px rgba(0, 0, 0, 0.2);

  #pdf-modal iframe {
      width: 100%;
      height: 100%;
    }
  }
</style>

<!-- JavaScript to control modal -->
<script>
  document.getElementById("open-modal").addEventListener("click", function(event) {
    event.preventDefault();
    document.getElementById("pdf-modal").style.display = "flex";
  });

  document.getElementById("close-modal").addEventListener("click", function() {
    document.getElementById("pdf-modal").style.display = "none";
  });
</script>

<!-- [Download Resume - PDF Version](https://github.com/abarton51/abarton51.github.io/blob/master/_files/Barton_Austin_T_Resume-2.pdf?raw=true){:target="_blank"} -->

{% include base_path %}

## Education
* (Current) B.S. in Mathematics and Computer Science, Georgia Institute of Technology, 2021-2025

## Research experience

* Georgia Tech Financial Services Innovation Lab, Atlanta, GA
  *  VIP Participant \| Jan, 2024 - May, 2025
      * Researched hawkish vs. dovish sentiment analysis using pretrained large language models (LLMs) on monetary policy reports from federal banks.
      * Reserached datasets for benchmarking LLMs' abilities to identify SEC violations given a scenario description.
      * Researched retrievel augmented generation (RAG) methods for knowledge intensive tasks for the financial domain.

* Dept. of Mathematics, North Carolina State University, Raleigh, NC
  * Researcher \| May, 2023 - Aug, 2023
      * Researched parameter estimation and modeling at the NSF and NSA sponsored research experience for undergraduates (REU) at NCSU.
      * Implemented Physics-Informed Neural Networks (PINNs) and novel equation learning techniques in Python using PyTorch to infer a system of differential equations for an agent-based model with adaptive behavior.

## Work experience

* Amazon Web Services (AWS), Bellevue, WA
  * Software Development Engineer Intern \| May, 2024 - Aug, 2024
      * Designed, implemented, and deployed an ETL (Extract, Transform, Load) data integration process for the Business Intelligence of an AWS service using cloud infrastructure for automated workflows, enhancing the reliability and accuracy of business metrics by integrating $\approx$35\% of previously unaccounted customer scenarios.
      * Implemented a robust dependency mocking framework, enabling the simulation of dependencies with dynamically generated test data, leading to more comprehensive integration tests.
      * Leveraged AWS Cloud Development Kit (CDK) as IaC to automate cloud, streamlining infrastructure processes.

* United States Marine Corps, Camp Lejeune, NC
  * Marine, Infantry Assultman (MOS 0351), Active Duty, Sergeant/E-5 \| Oct, 2016 - Oct, 2020
      * Led and collaborated with a team of 12 Marines in 2/6 Fox Co., 2/6 Marines to accomplish missions in hazardous environments.

## Skills

* **Programming Languages**:
  * _Highly Proficient in_: Python
  * _Proficient in_: Rust, C, Java, SQL
  * _Familiar with_: C++, Typescript, Javascript
* **Programming Libraries, Frameworks, etc.**:
  * PyTorch, Numpy, Sci-kit learn, Pandas, SQLAlchemy, Pydantic, SciPy, FastAPI, Scrapy, Langchain
* **Data Science/AI**: 
  * Deep Learning, Machine Learning (Unsupervised and Supervised), NLP, RAG, CV, Statistics
* **Core InfoSec Skills**: 
  * CIA Triad, AAA, Symmetric and Asymmetric Cryptography, Hashing, TLS/SSL, PKI, MitM & Replay Attacks
* **Exploits/Vulnerabilities**: 
  * Control Hijacking, Buffer Overflow & Overreads, XSS, CSRF, SQL Injection, Oracle Padding, Bleichenbacher
* **Database Systems**: 
  * PostgreSQL, MySQL
* **Web Development**: 
  * CRUD Applications, REST API, React, Node.js, HTML, CSS
* **Computer Systems**: 
  * GNU/Linux, Memory Hierarchy, Low-level debuggin (GDB), Build systems (Make, Cargo)
* **Computer Networking**: 
  * Network Protocols, Paket Sniffing (Wireshark)
* **Misc.**: 
  * VCS (Git), Containerization (Docker), Agile, CI/CD, IaC, Microservices, OOP/OOD, Data Structures, Algorithms
* **Cloud Computing**:
  * Amazon/AWS
    * DynamoDB, Lambda S3, Glue, IAM, Cloudwatch,Redshift, Step Functions, AWS CDK

## Projects

  <ul>{% for post in site.portfolio %}
   {% include archive-single.html %}
  {% endfor %}</ul>

