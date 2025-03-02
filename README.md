# Uploading a Local Repository to Hugging Face Hub
## Note
> This contains a guide on how upload your local directory to huggingface best used when you have model weights (as hugging face has no limitation to git lfs unline github which has 1 gb only) itll be very usefull


This script allows you to upload your local repository to the Hugging Face Hub. It copies all relevant files to a temporary directory and then uploads them to the specified Hugging Face repository.

## Prerequisites

Before running the script, ensure you have the following:

- A Hugging Face account
- A Hugging Face API token (can be generated in your account settings)
- The `huggingface_hub` Python package installed (`pip install huggingface_hub`)

## Configuration

Update the following variables in the script with your actual details:

```python
HF_USERNAME = "your_username"  # Replace with your Hugging Face username
HF_TOKEN = "your_huggingface_token"  # Replace with your API token
REPO_ID = "your_username/your_repo"  # Replace with your repository ID
```

## How the Script Works

1. Logs into Hugging Face using your API token.
2. Creates a temporary directory to stage files.
3. Copies all files from the local directory except `.git`.
4. Uploads the staged files to the specified Hugging Face repository.
> Why we exclude .git directory : its because hugginface sometimes say can continue because .git found or to keep the local git changes and huggingface changes separate  
## Running the Script

Run the script using Python:

```bash
python upload_to_huggingface.py
```

## Excluded Files

The script automatically excludes the following:

- `.git` directory

## Upload Confirmation

Once completed, you will see a confirmation message:

```bash
Upload complete.
Project uploaded to your_username/your_repo
```

## Code :
```python
from huggingface_hub import HfApi, login
import os
import shutil
import tempfile

# --- Configuration (REPLACE THESE WITH YOUR ACTUAL VALUES) ---
HF_USERNAME = "your_username"  # Your Hugging Face username
HF_TOKEN = "your_token"  # Create a token in your HF settings
REPO_ID = "username/repoid"  # Your repository ID (e.g., your_username/your_project_name)

login(token=HF_TOKEN)
api = HfApi()

# Create the repo if it doesn't exist (you might have already done this via the website)

# 1. Create a temporary directory to stage files for upload
with tempfile.TemporaryDirectory() as tmpdir:
    print(f"Staging files in temporary directory: {tmpdir}")
    # Walk through the project directory and copy files to the temp directory
    for root, _, files in os.walk("."):
        for file in files:
            file_path = os.path.join(root, file)
            relative_path = os.path.relpath(file_path, ".")

            # Exclude .git directory and hf.py and hf1.py files
            if ".git" not in file_path and file != "hf.py" and file != "hf1.py":
                source_path = file_path
                destination_path = os.path.join(tmpdir, relative_path)
                os.makedirs(os.path.dirname(destination_path), exist_ok=True) # Ensure dirs exist

                print(f"  Copying: {source_path} to {destination_path}") # Debug print

                shutil.copy2(source_path, destination_path) # copy with metadata

                # Debug: Verify directory creation in temp dir
                dest_dir = os.path.dirname(destination_path)
                if os.path.exists(dest_dir) and os.path.isdir(dest_dir):
                    print(f"  DEBUG: Directory '{dest_dir}' EXISTS in temp dir.")
                else:
                    print(f"  DEBUG: Directory '{dest_dir}' DOES NOT EXIST in temp dir! (Problem?)")


    # 2. Upload the temporary directory using upload_folder
    print("Uploading staged files to Hugging Face Hub...")
    api.upload_folder(
        repo_id=REPO_ID,
        folder_path=tmpdir,  # Upload the entire temp directory
        path_in_repo=".",      # Upload to the root of the repo, preserving structure
    )

print("Upload complete.")
print(f"Project uploaded to {REPO_ID}")
```

## This is the fast and easier way to upload if your in a hurry 
## I tried using git to upload but its hard to manage when you have files with size more 10MB

---

## Uploading a Local Repository Using Git

> Use this when you want upload mostly code 
Alternatively, you can upload your local repository to Hugging Face Hub using Git by following these steps:

### Prerequisites
- Install Git (`git` should be installed on your system).
- Authenticate with Hugging Face CLI:

```bash
huggingface-cli login
```

This will prompt you to enter your Hugging Face API token.

### Steps to Upload
1. Initialize a Git repository (if not already initialized):
   ```bash
   git init
   ```
2. Add the Hugging Face repository as a remote:
   ```bash
   git remote add origin https://huggingface.co/your_username/your_repo.git
   ```
3. Add and commit your files:
   ```bash
   git add .
   git commit -m "Initial commit"
   ```
4. Push the repository to Hugging Face:
   ```bash
   git push origin main
   ```

Your repository will now be available on the Hugging Face Hub.


### Steps to Upload Model Weights

1. Track large files using Git LFS:
   ```bash
   git lfs track "*.bin"
   git lfs track "*.pt"
   ```
> or you could give give git lfs track `path/model_weights/*` to track all the files in the path or `path/model_weights/*.ckpt` to only track files with this extension

> To track a full folder you can type
```
git lfs track "dir_name/**"
```
> and this will make git lfs track the full folder
2. Add the Hugging Face repository as a remote:
   ```bash
   git remote add origin https://huggingface.co/your_username/your_repo.git
   ```
3. Add and commit your files:
   ```bash
   git add .gitattributes *.bin *.pt
   git commit -m "Add model weights"
   ```
4. Push the repository to Hugging Face:
   ```bash
   git push origin main
   ```

Your model weights will now be stored and managed using Git LFS on the Hugging Face Hub.
