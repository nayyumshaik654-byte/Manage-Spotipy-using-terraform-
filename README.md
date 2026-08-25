# Manage-Spotipy-using-terraform-
In this hands on devops project we use Terraform to create a Spotify playlist using Infrastructure as code principle. This project will teach you many terraform concepts including provider, data blocks, how to write terraform code, define variable and more.
Prerequisites :
Terraform (v1.0+) installed on your machine.Docker
Desktop installed and running (used to run the local OAuth proxy for authentication).
A Spotify Account (Free or Premium).
A Spotify Developer Account.

Step 1: Spotify Developer Dashboard SetupLog in to the Spotify Developer Dashboard.
Click Create app.
Fill in the app name and description.
Set the Redirect URI to http://localhost:27228/callback and click Add, 
then save your changes.
Copy your Client ID and Client Secret.

Step 2: Run the Authentication ProxyTerraform needs an authentication token to speak to Spotify. 
Run the official authorization proxy server locally via
"Docker:bashdocker run --rm -it -p 27228:27228 -e SPOTIFY_CLIENT_ID=your_client_id -e SPOTIFY_CLIENT_SECRET=your_client_secret conradludgate/spotify-auth-proxy"
Use code with caution.(Open the generated link in your browser to log in and authorize the app.)

Step 3: Terraform Configuration FilesCreate a project directory and add the following : 
files:provider.tf 
 hclterraform {  
  required_providers {
    spotify = {
      source  = "conradludgate/spotify"
      version = "~> 0.2.0"
    }
  }
}

provider "spotify" {
  # Authenticates against the local Docker auth proxy
  api_key = var.spotify_api_key
}

file : variables.tf 
hclvariable "spotify_api_key" {
  type        = string
  description = "The Spotify authorization token/API key"
  sensitive   = true
}

 file : terraform.tfvars
spotify_api_key = "your_generated_api_key_from_proxy"

 file : main.tf
 resource "spotify_playlist" "my_fav_playlist" {
  name        = "Terraform Awesome Mix"
  description = "Managed entirely via Infrastructure as Code!"
  public      = false

  tracks = [ data.spotify_track.track_1.id, 
  data.spotify_track.track_2.id,
  ]
}

data "spotify_track" "track_1" {
  url = "https://open.spotify.com/track/4XdaaDFE881SlIaz31pTAG"
}

data "spotify_track" "track_2" {
  url = "https://open.spotify.com/track/4lE6N1E0L8CssgKEUCgdbA"
}

Step 4: Run Terraform CommandsInitialize the provider and apply the configuration:
bash
terraform init
terraform plan
terraform apply



