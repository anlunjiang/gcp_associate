* You are creating a Google Cloud account for your own personal projects. Select the correct account type.
    * Individual
* When would you need to use Google Cloud Directory Sync?
    * When adding users from Gmail.
* What best describes the project ID?
    * A globally-unique identifier used by engineers that we can set optionally ourselves.
* What budget alerts are preset by Google?
    * 50%, 90%, 100%
* You need multiple Cloud SDK components to manage services for your project. Which of the following Cloud SDK components will need to be installed before you can use it because it is not pre-installed?
    * alpha
* The gcloud init command is a shortcut for which two commands?
    * The login and config commands
* You are installing Cloud SDK on an Ubuntu VM. Describe the required commands that need to run to install Cloud SDK.
    * Add the Cloud SDK distribution URI as a package source
    * Import the Google Cloud Platform public key
    * Update the package list and install the Cloud SDK
* Additional groups of Cloud SDK commands that you can install for the gcloud component are __ commands.
    * Alpha and Beta
* What is an advantage of using a flexible environment with App Engine?
    * Can customize the runtime
* Which controller uses desired state configuration and allows us to specify the number of pod instances running on a cluster?
    * Deployment
* Which of the following statements are true regarding managed instance groups?
    * Additional instances are removed 10 minutes after they are no longer required.
    * They offer high availability, autoscaling, and automatic updates.
    * They are suitable for stateless applications.
* After creating your Kubernetes cluster, what needs to be done to interact with the Kubernetes cluster?
    * Run gcloud container clusters get-credentials to authenticate and configure kubectl.
* Which of the following commands will set the default project?
    * gcloud config set project [PROJECT_ID]
* Which command is used to create a Google Cloud Function?
    * gcloud functions deploy
* According to Application Default Credentials (ADC), how does a Google Cloud client library locate application credentials?
    * First, ADC searches for environment variable GOOGLE_APPLICATION_CREDENTIALS to see if the service account key is stored.
    * If GOOGLE_APPLICATION_CREDENTIALS is not set, then ADC will use the default service account. If the default service account cannot be used, then there will be an error, and you will need to provide the credentials.
* Which command is used to create a storage bucket for Cloud Storage?
    * gsutil mb
* Which command is used to copy to or from a Cloud Storage bucket?
    * gsutil cp
* Kubernetes works with _ files to specify the resources that we need.
    * YAML
* What command is used to create a Spanner database table?
    * gcloud spanner databases ddl
* What is a way to reduce network latency when creating a Kubernetes cluster?
    * Use the same zone as the other services
* How are access scopes used?
    * Access scopes are a legacy predating Cloud IAM. They are used to assign permissions to VM instances.
* A point-in-time copy of a persistent disk is called a:
    * Snapshot
* Which of the following statements are true concerning static IP addresses? Select the most appropriate response.
    * With a static internal IP address, we can specify the IP address or let Google select an IP address from the range.
    * With a static external IP address, we cannot specify the IP address. Google selects an IP address from the range.
* What action is performed by the gcloud config list command?
    * List the settings for the active configuration
* Log sinks can export records to which of the following destinations?
    * Cloud Pub/Sub
* Which type of scaling can NOT be used by flexible environments?
    * Basic
* What action is performed by the gcloud config configurations create command
    * Create and activate a new configuration
* Which ways can we can split traffic in App Engine?
    * Cookie
    * Random
    * IP Address