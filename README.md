# Terraform Google Cloud Platform Event Cloud Function

Terraform deploy a local folder to a Google Cloud Function that can be triggered using an Event

## Basic Usage

```hcl
# Create a Storage Bucket to store the Cloud Functions in
resource "google_storage_bucket" "cloudfunctions_bucket" {
  name     = "myproject-cloud-functions"
  location = "europe-west3"
}

# Create a cloud function named `hello-world`
module "cloudfunction--hello-world" {
  source          = "github.com/bramus/terraform-gcloud-event-cloud-function"
  bucket_name     = google_storage_bucket.cloudfunctions_bucket.name
  name            = "hello-world"
  event_resource  = "projects/my-project/topics/my-topic"
}
```

The module `terraform-gcloud-event-cloud-function` will:

1. Zip up the contents of the `./cloudfunctions/hello-world` folder
2. Store the `hello-world.zip` file as an object into the bucket
3. Create the Cloud Function, linking to the `hello-world.zip` object in the bucket
4. Configure the Cloud Function to be invoked whenever a Pub/Sub message gets published on the `"projects/my-project/topics/my-topic"` Topic

## Variables

### Required Variables

- `bucket_name`: Name of GCS bucket to use to store the Cloud Functions their contents on.
- `name`: Name of the cloud function
- `event_resource`: The name or partial URI of the resource from which to observe events. For example, `"myBucket"` or `"projects/my-project/topics/my-topic"`

### Optional Variables and their defaults

- `source_dir`= `"./cloudfunctions/${var.name}"`
- `description`: `"${var.name} Event Cloud Function"`
- `runtime`: `"python39"` _(One of [https://cloud.google.com/functions/docs/concepts/exec#runtimes](https://cloud.google.com/functions/docs/concepts/exec#runtimes))_
- `entry_point`: `"__main__"`
- `available_memory_mb`: `128`
- `timeout`: `60`
- `environment_variables`= `{}`
- `service_account_email` = `""`
- `vpc_connector` = `null`
- `max_instances` = `null`
- `event_type` = `"google.pubsub.topic.publish"` _(One of [https://cloud.google.com/functions/docs/calling/#event_triggers](https://cloud.google.com/functions/docs/calling/#event_triggers))_

## Extended Example (Overriding the defaults)

```hcl
# Create a Storage Bucket to store the Cloud Functions in
resource "google_storage_bucket" "cloudfunctions_bucket" {
  name     = "myproject-cloud-functions"
  location = "europe-west3"
}

# Create a cloud function named `postprocess-bucket-upload`
module "cloudfunction--postprocess-bucket-upload" {
  source                 = "github.com/bramus/terraform-gcloud-event-cloud-function"
  bucket_name            = google_storage_bucket.cloudfunctions_bucket.name
  event_trigger          = "google.storage.object.finalize"
  event_resource         = "name-of-some-bucket-to-watch"
  name                   = "postprocess-bucket-upload"
  source_dir             = "./functions/postprocess-bucket-upload/src"
  runtime                = "php74"
  entry_point            = "postprocessbucketuploadPubsub"
  available_memory_mb    = 256
  timeout                = 120
  service_account_email  = "cloud-function-invoker@project.iam.gserviceaccount.com"
  vpc_connector          = "vpc-access-connector-name"
  max_instances          = 200
}
```

## License

`terraform-gcloud-event-cloud-function` is released under the MIT License. See the enclosed [`LICENSE` file](LICENSE) for details.