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



















Create a Spotify playlist with Terraform
15min
|
Terraform
Video

Reference this often? Create an account to bookmark tutorials.

Terraform manages infrastructure on cloud computing providers such as AWS, Azure, and GCP. But, it can also manage resources in hundreds of other services, including the music service Spotify.

In this tutorial, you will use a Terraform data source to search Spotify for an artist, album, or song, and use that data to build a playlist.

Prerequisites
To complete this tutorial, you will need:

Terraform version 1.0+
Docker Desktop
Spotify account with developer access
Create Spotify developer app
Before you can use Terraform with Spotify, you need to create a Spotify developer app and run Spotify's authorization proxy server.

Login to the Spotify developer dashboard.

Spotify developer dashboard

Click the green Create an app button.

Fill out the name and description according to the table below, check the box to agree to the terms of services, then click Create.

Name	Description
Terraform Playlist Demo	Create a Spotify playlist using Terraform. Follow the tutorial at learn.hashicorp.com/tutorials/terraform/spotify-playlist
Create an app in the Spotify developer dashboard

Once Spotify creates the application, find and click the green Edit Settings button on the top right side.

Copy the URI below into the Redirect URI field and click Add so that Spotify can find its authorization application locally on port 27228 at the correct path. Scroll to the bottom of the form and click Save.

http://localhost:27228/spotify_callback

Updated Redirect URI for Spotify application

Run authorization server
Now that you created the Spotify app, you are ready to configure and start the authorization proxy server, which allows Terraform to interact with Spotify.

Diagram of the authorization flow through the Spotify authorization
proxy

Return to your terminal and set the redirect URI as an environment variable, instructing the authorization proxy server to serve your Spotify access tokens on port 27228.

$ export SPOTIFY_CLIENT_REDIRECT_URI=http://localhost:27228/spotify_callback

Next, create a file called .env with the following contents to store your Spotify application's client ID and secret.

SPOTIFY_CLIENT_ID=
SPOTIFY_CLIENT_SECRET=

Copy the Client ID from the Spotify app page underneath your app's title and description, and paste it into .env as your SPOTIFY_CLIENT_ID.

Spotify Developer portal app page with the client ID and secret

Click Show client secret and copy the value displayed into .env as your SPOTIFY_CLIENT_SECRET.

Make sure Docker Desktop is running, and start the server. It will run in your terminal's foreground.

$ docker run --rm -it -p 27228:27228 --env-file ./.env ghcr.io/conradludgate/spotify-auth-proxy
Unable to find image 'ghcr.io/conradludgate/spotify-auth-proxy:latest' locally
latest: Pulling from conradludgate/spotify-auth-proxy
5843afab3874: Pull complete
b244520335f6: Pull complete
Digest: sha256:c738f59a734ac17812aae5032cfc6f799e03c1f09d9146edb9c2836bc589f3dc
Status: Downloaded newer image for ghcr.io/conradludgate/spotify-auth-proxy:latest
APIKey: xxxxxx...
Token:  xxxxxx...
Auth:   http://localhost:27228/authorize?token=xxxxxx...

Visit the authorization server's URL by visiting the link that your terminal output lists after Auth:.

The server will redirect you to Spotify to authenticate. After authenticating, the server will display Authorization successful, indicating that the Terraform provider can use the server to retrieve access tokens.

Leave the server running.

Clone example repository
Clone the example Terraform configuration for this tutorial. It contains a complete Terraform configuration that searches for songs by Dolly Parton, and creates a playlist out of them.

$ git clone https://github.com/hashicorp-education/learn-terraform-spotify

Change into the directory.

$ cd learn-terraform-spotify

Explore the configuration
Open main.tf. This file contains the Terraform configuration that searches Spotify and creates the playlist. The first two configuration blocks in the file:

configure Terraform itself and specify the community provider that Terraform uses to communicate with Spotify.
configure the Spotify provider with the key you set as a variable.
terraform {
  required_providers {
    spotify = {
      version = "~> 0.1.5"
      source  = "conradludgate/spotify"
    }
  }
}

provider "spotify" {
  api_key = var.spotify_api_key
}

The next block defines a Terraform data source to search the Spotify provider for Dolly Parton songs.

data "spotify_search_track" "by_artist" {
  artists = ["Dolly Parton"]
  #  album = "Jolene"
  #  name  = "Early Morning Breeze"
}

The next block uses a Terraform resource to create a playlist from the first three songs that match the search in the data source block.

resource "spotify_playlist" "playlist" {
  name        = "Terraform Summer Playlist"
  description = "This playlist was created by Terraform"
  public      = true

  tracks = [
    data.spotify_search_track.by_artist.tracks[0].id,
    data.spotify_search_track.by_artist.tracks[1].id,
    data.spotify_search_track.by_artist.tracks[2].id,
  ]
}

Open outputs.tf, which defines an output value for the URL of the playlist.

output "playlist_url" {
  value       = "https://open.spotify.com/playlist/${spotify_playlist.playlist.id}"
  description = "Visit this URL in your browser to listen to the playlist"
}

Set the API key
Rename the terraform.tfvars.example file terraform.tfvars so that Terraform can detect the file.

$ mv terraform.tfvars.example terraform.tfvars

The .gitignore file in this repository excludes files with the .tfvars extension from version control to prevent you from accidentally committing your credentials.

Warning

Never commit sensitive values to version control.

Find the terminal window where the Spotify authorization proxy server is running and copy the APIKey from its output.

Open terraform.tfvars, and replace ... with the key from the proxy, so that Terraform can authenticate with Spotify. Save the file.

spotify_api_key = "..."

This variable is declared for you in variables.tf.

variable "spotify_api_key" {
  type        = string
  description = "Set this as the APIKey that the authorization proxy server outputs"
}

Install the Spotify provider
In your terminal, initialize Terraform, which will install the Spotify provider.

$ terraform init

Initializing the backend...

Initializing provider plugins...
- Finding conradludgate/spotify versions matching "~> 0.1.5"...
- Installing conradludgate/spotify v0.1.5...
- Installed conradludgate/spotify v0.1.5 (self-signed, key ID B4E4E68AFAC5D89C)

Partner and community providers are signed by their developers.
If you'd like to know more about provider signing, you can read about it here:
https://www.terraform.io/docs/cli/plugins/signing.html

Terraform has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.

Create the playlist
Now you are ready to create your playlist. Apply your Terraform configuration. Terraform will show you the changes it plans to make and prompt for your approval.

$ terraform apply

Terraform used the selected providers to generate the following execution plan. Resource actions are
indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # spotify_playlist.playlist will be created
  + resource "spotify_playlist" "playlist" {
      + description = "This playlist was created by Terraform"
      + id          = (known after apply)
      + name        = "Terraform Summer Playlist"
      + public      = true
      + snapshot_id = (known after apply)
      + tracks      = [
          + "2SpEHTbUuebeLkgs9QB7Ue",
          + "4w3tQBXhn5345eUXDGBWZG",
          + "6dnco8haegnJYtylV26cBq",
        ]
    }

Plan: 1 to add, 0 to change, 0 to destroy.

Changes to Outputs:
  + playlist_url = (known after apply)

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value:

Confirm the apply with a yes, and Terraform will create your playlist.

  Enter a value: yes

spotify_playlist.playlist: Creating...
spotify_playlist.playlist: Creation complete after 1s [id=40bGNifvqzwjO8gHDvhbB3]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.

Outputs:

playlist_url = "https://open.spotify.com/playlist/40bGNifvqzwjO8gHDvhbB3"

Listen to your playlist
Open the playlist URL returned in the Terraform output and enjoy your playlist!

Spotify playlist created with Terraform

Customize and share!
Now that you have created a playlist using Terraform, build your own Spotify playlist. Terraform maintains a state file that allows it to manage the playlist you created. Change the playlist name, the artist, specify an album, or pick specific tracks with their Spotify song IDs or URLs. See the example below for inspiration.

# Optional: Find song by ID
data "spotify_track" "early_morning_breeze_by_id" {
  spotify_id = "3CkJeQjxbYJsLN2MLAvluL"
}
# Optional: Find song by URL
data "spotify_track" "early_morning_breeze_by_url" {
  url = "https://open.spotify.com/track/3CkJeQjxbYJsLN2MLAvluL?si=753e257f39ac4b45"
}

When you are happy with your configuration, apply your changes to edit the playlist, and remember to confirm the plan that Terraform presents with a yes.

$ terraform apply

Explore the unofficial Spotify provider documentation for all the available configuration options.

Share your work!
Share the playlist on Twitter or LinkedIn with the hashtag #TerraformSpotify. Please adhere to our community guidelines and manually scrub explicit songs from your playlist before sharing.

Here is a tweet you can customize with your playlist ID. Happy hacking!

Check out the Spotify playlist that I built with Terraform!
https://open.spotify.com/playlist/YOUR_PLAYLIST_ID

Follow this tutorial to build your own.
https://hashi.co/terraform-spotify-playlist

#TerraformSpotify @HashiCorp @Spotify

Next step



