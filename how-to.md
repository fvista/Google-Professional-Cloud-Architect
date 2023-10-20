# How-To

## Create a persistent state in Cloud Shell

The gcloud command often requires you to specify values such as a Region, Zone, or Project ID. Entering them repeatedly increases the chance of making typing errors. If you use Cloud Shell frequently, you may want to set common values in environment variables and use them instead of typing the actual values.

1. Type in the shell

```
INFRACLASS_REGION=[YOUR_REGION]
INFRACLASS_PROJECT_ID=[YOUR_PROJECT_ID]
```

Note: Every time you close Cloud Shell and reopen it, a new VM is allocated, and the environment variable you just set disappears. 

```
mkdir infraclass
touch infraclass/config
echo INFRACLASS_REGION=$INFRACLASS_REGION >> ~/infraclass/config
echo INFRACLASS_PROJECT_ID=$INFRACLASS_PROJECT_ID >> ~/infraclass/config
source infraclass/config
```
2. Modify the bash profile and create persistence

``` 
nano .profile 
```

3. Add the following line to the end of the file:

``` 
source infraclass/config
```

Note: If your Cloud Shell environment is ever corrupted, instructions on resetting it are in the Cloud Shell Documentation (https://cloud.google.com/shell/docs/resetting-cloud-shell) article titled Disabling or Resetting Cloud Shell. This will cause everything in your Cloud Shell environment to be set back to its original default state.

## Create a VPC network with firewall rules and two VM instances

Google Cloud Virtual Private Cloud (VPC) offers networking capabilities for Compute Engine virtual machine (VM) instances, Kubernetes Engine containers, and the App Engine flexible environment. In simpler terms, VPC is essential for creating VM instances, containers, or App Engine applications. Consequently, every Google Cloud project includes a default network to help you begin your setup.


![alt text](https://cdn.qwiklabs.com/lX%2BlZ11ZdTwmfFe3Kwrh3uu3mWYFQwnS0LdcgBS70ng%3D)

### Commands for Network Creation

- Create network and subnetwork managementnet

```bash
gcloud compute networks create managementnet --project=qwiklabs-gcp-04-a11433ba4dc7 --subnet-mode=custom --mtu=1460 --bgp-routing-mode=regional

gcloud compute networks subnets create managementsubnet-us --project=qwiklabs-gcp-04-a11433ba4dc7 --range=10.240.0.0/20 --stack-type=IPV4_ONLY --network=managementnet --region=us-west1
```

- Create network and subnetwork privatenet 

```bash
gcloud compute networks create privatenet --subnet-mode=custom

gcloud compute networks subnets create privatesubnet-us --network=privatenet --region=us-west1 --range=172.16.0.0/24

gcloud compute networks subnets create privatesubnet-eu --network=privatenet --region=europe-west1 --range=172.20.0.0/20
```

- Create a firewall-rule
```bash
gcloud compute --project=qwiklabs-gcp-04-a11433ba4dc7 firewall-rules create managementnet-allow-icmp-ssh-rdp --direction=INGRESS --priority=1000 --network=managementnet --action=ALLOW --rules=tcp:22,tcp:3389,icmp --source-ranges=0.0.0.0/0
```

- Create VM instances
```bash
gcloud compute instances create managementnet-us-vm \
    --project=qwiklabs-gcp-04-a11433ba4dc7 \
    --zone=us-west1-c \
    --machine-type=e2-micro \
    --network-interface=network-tier=PREMIUM,stack-type=IPV4_ONLY,subnet=managementsubnet-us \
    --metadata=enable-oslogin=true \
    --maintenance-policy=MIGRATE \
    --provisioning-model=STANDARD \
    --service-account=1094778087533-compute@developer.gserviceaccount.com \
    --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append \
    --create-disk=auto-delete=yes,boot=yes,device-name=managementnet-us-vm,image=projects/debian-cloud/global/images/debian-11-bullseye-v20231010,mode=rw,size=10,type=projects/qwiklabs-gcp-04-a11433ba4dc7/zones/us-west1-c/diskTypes/pd-balanced \
    --no-shielded-secure-boot \
    --shielded-vtpm \
    --shielded-integrity-monitoring \
    --labels=goog-ec-src=vm_add-gcloud \
    --reservation-affinity=any
```

- or equivalent Terraform HCL: 
```Terraform
# This code is compatible with Terraform 4.25.0 and versions that are backwards compatible to 4.25.0.
# For information about validating this Terraform code, see https://developer.hashicorp.com/terraform/tutorials/gcp-get-started/google-cloud-platform-build#format-and-validate-the-configuration

resource "google_compute_instance" "managementnet-us-vm" {
  boot_disk {
    auto_delete = true
    device_name = "managementnet-us-vm"

    initialize_params {
      image = "projects/debian-cloud/global/images/debian-11-bullseye-v20231010"
      size  = 10
      type  = "pd-balanced"
    }

    mode = "READ_WRITE"
  }

  can_ip_forward      = false
  deletion_protection = false
  enable_display      = false

  labels = {
    goog-ec-src = "vm_add-tf"
  }

  machine_type = "e2-micro"

  metadata = {
    enable-oslogin = "true"
  }

  name = "managementnet-us-vm"

  network_interface {
    access_config {
      network_tier = "PREMIUM"
    }

    subnetwork = "projects/qwiklabs-gcp-04-a11433ba4dc7/regions/us-west1/subnetworks/managementsubnet-us"
  }

  scheduling {
    automatic_restart   = true
    on_host_maintenance = "MIGRATE"
    preemptible         = false
    provisioning_model  = "STANDARD"
  }

  service_account {
    email  = "XXX@XXX"
    scopes = ["https://www.googleapis.com/auth/devstorage.read_only", "https://www.googleapis.com/auth/logging.write", "https://www.googleapis.com/auth/monitoring.write", "https://www.googleapis.com/auth/service.management.readonly", "https://www.googleapis.com/auth/servicecontrol", "https://www.googleapis.com/auth/trace.append"]
  }

  shielded_instance_config {
    enable_integrity_monitoring = true
    enable_secure_boot          = false
    enable_vtpm                 = true
  }

  zone = "us-west1-c"
}
```