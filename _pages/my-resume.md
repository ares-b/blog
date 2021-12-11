---
layout: page
title: My Resume
permalink: /resume
comments: false
---
{% assign author = site.authors['ares'] %}
<div class="row justify-content-between">
<div class="col-md-4">

<div class="sticky-top sticky-top-80">
    <!-- Author Box -->
    {% include author.html %}

    <h5>Contact</h5>
    <p class="resume">Phone number  : +33 7 69 75 53 13</p>
    <p class="resume">Email         : <a href="mailto:arslane.bahlel@gmail.com">arslane.bahlel@gmail.com</a></p>
    <h5>Languages</h5>
    <p class="resume">English       : Professional</p>
    <p class="resume">French        : Fluent</p>
    <p class="resume">Arabic        : Basic</p>
    <h5>Skills</h5>
    <p class="resume">Languages               : Scala, Java & Python</p>
    <p class="resume">Data Storage            : Hadoop(CDH & MapR), Elasticsearch, SQLServer & PostgreSQL</p>
    <p class="resume">Data Processing         : Spark & Akka</p>
    <p class="resume">Flow Orchestration      : Airflow & Oozie</p>
    <p class="resume">CI/CD                   : Gitlab CI/CD, Jenkins & Github Actions</p>
    <p class="resume">Designing               : Design Patterns, Lambda & Kappa architectures</p>
    <h5>Certifications</h5>
    <p class="resume"><a href="https://www.credly.com/badges/68da7859-5a11-46e3-8bee-e1aaaa7be31c">Apache Airflow Fundamentals</a></p>
    <p class="resume"><a href="https://www.coursera.org/account/accomplishments/verify/KLRDCN93N8FX">EPFL's Functional Programming Principles in Scala</a></p>

</div>
</div>
<div class="col-md-8 pr-5">
<h5>Work Experience</h5>
    <div class="resume-company">
        <div id="company" class="row justify-content-between">
            <div class="col-md-3">Nov. 2021 - Present</div>
            <div class="col-md-9"><h4>Data Engineer Consultant - Capgemini</h4></div>
        </div>
        <div id="mission-caas">
            <div id="mission-description" class="row justify-content-between">
                <div class="col-md-3 resume-logo-container"><img class="resume-logo" src="/assets/images/resume/logo-caas.png"/></div>
                <div class="col-md-9">
                    <h5>Big Data Engineer at Crédit Agricle Assurances</h5>
                    <p class="resume">Coming Soon</p>
                </div>
        </div>
        </div>
    </div>
    <div class="resume-company">
        <div id="company" class="row justify-content-between">
            <div class="col-md-3">Nov. 2020 - Sept. 2021</div>
            <div class="col-md-9"><h4>Apprentice Data Engineer - Numberly</h4></div>
        </div>
        <div id="mission-ikks">
            <div id="mission-description" class="row justify-content-between">
                <div class="col-md-3 resume-logo-container"><img class="resume-logo" src="/assets/images/resume/logo-ikks.png"/></div>
                <div class="col-md-9">
                    <h5>Big Data Engineer at IKKS</h5>
                        <p>IKKS wanted to set up a customer data platform (CDP) to enrich its knowledge of its customers</p>
                        <ul>
                            <li><p class="resume">Developped pipelines for marketing data flow integration and processing</p></li>
                            <li><p class="resume">Developped CI/CD pipelines in Gitlab</p></li>
                            <li><p class="resume">Developped a matching algorithm to uniquely identify an IKKS customer with multiple enties into the database</p></li>
                            <li><p class="resume">Implemented data restitution flow (Bounces, etc)</p></li>
                        </ul>
                </div>
            </div>
            <div id="technical-env" class="row justify-content-between">
                <div class="col-md-12">
                    <p class="resume">
                        <b>Technical envirnment</b> : Python, Airflow, Spark, Hive, HDFS (CDH) & SQLServer 
                    </p>
                </div>
            </div>
        </div>
        <div id="mission-crma">
            <div id="mission-description" class="row justify-content-between">
                <div class="col-md-3 resume-logo-container"><img class="resume-logo" src="/assets/images/resume/logo-numberly.png"/></div>
                <div class="col-md-9">
                    <h5>Big Data Engineer at Numberly</h5>
                        <ul>
                            <li><p class="resume">Developped pipelines for fake customer data generation (for Q&A phases)</p></li>
                            <li><p class="resume">Developped CI/CD pipelines in Gitlab</p></li>
                        </ul>
                </div>
            </div>
            <div id="technical-env" class="row justify-content-between">
                <div class="col-md-12">
                    <p class="resume">
                        <b>Technical envirnment</b> : Python, Airflow, Spark, Hive, HDFS (CDH) & SQLServer 
                    </p>
                </div>
            </div>
        </div>
    </div>
    <div class="resume-company">
        <div id="company" class="row justify-content-between">
            <div class="col-md-3">Nov. 2021 - Present</div>
            <div class="col-md-9"><h4>Appentice Data Engineer/Scientist - Capgemini</h4></div>
        </div>
        <div id="mission-officedepot">
            <div id="mission-description" class="row justify-content-between">
                <div class="col-md-3 resume-logo-container"><img class="resume-logo" src="/assets/images/resume/logo-officedepot.png"/></div>
                <div class="col-md-9">
                    <h5>Data Engineer at Office Dépôt France</h5>
                    <p>Overhaul of Office Dépôt's Information system</p>
                    <ul>
                        <li><p class="resume">Developped pipelines for integration SAP data into Salesforces and vice-versa</p></li>
                        <li><p class="resume">Monitored all Mulesoft applications (over 20)</p></li>
                    </ul>
                </div>
            </div>
            <div id="technical-env" class="row justify-content-between">
                <div class="col-md-12">
                    <p class="resume">
                        <b>Technical envirnment</b> : Mulesoft, Data Weave, Java & RAML
                    </p>
                </div>
            </div>
        </div>
        <div id="mission-pif">
            <div id="mission-description" class="row justify-content-between">
                <div class="col-md-3 resume-logo-container"><img class="resume-logo" src="/assets/images/resume/logo-capgemini.png"/></div>
                <div class="col-md-9">
                    <h5>Lead Data Engineer at Capgemini</h5>
                    <p>Prediction of webserver incidents (JIRA)</p>
                    <ul>
                        <li><p class="resume">Developed and improved incident prediction models using Machine Learning</p></li>
                        <li><p class="resume">Developed dashboards for data visualization and the prediction results</p></li>
                        <li><p class="resume">Developed Data Augmentation techniques</p></li>
                    </ul>
                </div>
            </div>
            <div id="technical-env" class="row justify-content-between">
                <div class="col-md-12">
                    <p class="resume">
                        <b>Technical envirnment</b> : Python, Spark, Elasticsearch & Dataiku
                    </p>
                </div>
            </div>
        </div>
    </div>

</div>


</div>
