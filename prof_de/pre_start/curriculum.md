Google Cloud Platform - Professional Data Engineer Certification

Curriculum:
1. Designing data processing systems
    * Selecting the appropriate storage technologies. Considerations include:
        * Mapping storage systems to business requirements
        * Data modeling
        * Tradeoffs involving latency, throughput, transactions
        * Distributed systems
        * Schema design
    * Designing data pipelines. Considerations include:
        * Data publishing and visualization (e.g., BigQuery)
        * Batch and streaming data (e.g., Cloud Dataflow, Cloud Dataproc, Apache Beam, Apache Spark and Hadoop ecosystem, Cloud Pub/Sub, Apache Kafka)
        * Online (interactive) vs. batch predictions
        * Job automation and orchestration (e.g., Cloud Composer)
    * Designing a data processing solution. Considerations include:
        * Choice of infrastructure
        * System availability and fault tolerance
        * Use of distributed systems
        * Capacity planning
        * Hybrid cloud and edge computing
        * Architecture options (e.g., message brokers, message queues, middleware, service-oriented architecture, serverless functions)
        * At least once, in-order, and exactly once, etc., event processing
    * Migrating data warehousing and data processing. Considerations include:
        * Awareness of current state and how to migrate a design to a future state
        * Migrating from on-premises to cloud (Data Transfer Service, Transfer Appliance, Cloud Networking)
        * Validating a migration
2. Building and operationalizing data processing systems    
    * Building and operationalizing storage systems. Considerations include:
        * Effective use of managed services (Cloud Bigtable, Cloud Spanner, Cloud SQL, BigQuery, Cloud Storage, Cloud Datastore, Cloud Memorystore)
        * Storage costs and performance
        * Lifecycle management of data
    * Building and operationalizing pipelines. Considerations include:
        * Data cleansing
        * Batch and streaming
        * Transformation
        * Data acquisition and import
        * Integrating with new data sources
    * Building and operationalizing processing infrastructure. Considerations include:
        * Provisioning resources
        * Monitoring pipelines
        * Adjusting pipelines
        * Testing and quality control
3. Operationalizing machine learning models
    * Leveraging pre-built ML models as a service. Considerations include:
        * ML APIs (e.g., Vision API, Speech API)
        * Customizing ML APIs (e.g., AutoML Vision, Auto ML text)
        * Conversational experiences (e.g., Dialogflow)
    * Deploying an ML pipeline. Considerations include:
        * Ingesting appropriate data
        * Retraining of machine learning models (Cloud Machine Learning Engine, BigQuery ML, Kubeflow, Spark ML)
        * Continuous evaluation
    * Choosing the appropriate training and serving infrastructure. Considerations include:
        * Distributed vs. single machine
        * Use of edge compute
        * Hardware accelerators (e.g., GPU, TPU)
    * Measuring, monitoring, and troubleshooting machine learning models. Considerations include:
        * Machine learning terminology (e.g., features, labels, models, regression, classification, recommendation, supervised and unsupervised learning, evaluation metrics)
        * Impact of dependencies of machine learning models
        * Common sources of error (e.g., assumptions about data)
4. Ensuring solution quality
    * Designing for security and compliance. Considerations include:
        * Identity and access management (e.g., Cloud IAM)
        * Data security (encryption, key management)
        * Ensuring privacy (e.g., Data Loss Prevention API)
        * Legal compliance (e.g., Health Insurance Portability and Accountability Act (HIPAA), Children's Online Privacy Protection Act (COPPA), FedRAMP, General Data Protection Regulation (GDPR))
    * Ensuring scalability and efficiency. Considerations include:
        * Building and running test suites
        * Pipeline monitoring (e.g., Stackdriver)
        * Assessing, troubleshooting, and improving data representations and data processing infrastructure
        * Resizing and autoscaling resources
    * Ensuring reliability and fidelity. Considerations include:
        * Performing data preparation and quality control (e.g., Cloud Dataprep)
        * Verification and monitoring
        * Planning, executing, and stress testing data recovery (fault tolerance, rerunning failed jobs, performing retrospective re-analysis)
        * Choosing between ACID, idempotent, eventually consistent requirements
    * Ensuring flexibility and portability. Considerations include:
        * Mapping to current and future business requirements
        * Designing for data and application portability (e.g., multi-cloud, data residency requirements)
        * Data staging, cataloging, and discovery