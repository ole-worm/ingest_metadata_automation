#!/bin/bash

# Function to replace illegal characters in a string
replace_illegal_chars() {
  local str=$1
  str=$(echo "$str" | tr -d '/\<>:|?*')
  echo "$str"
}

# Function to prompt for revision
prompt_revision() {
  local prompt_msg=$1
  local original_value=$2

  while true; do
    read -p "$prompt_msg [y/n] " -r
    echo

    if [[ $REPLY =~ ^[Yy]$ ]]; then
      read -p "Press Enter" revised_value
      echo
      revised_value=$(replace_illegal_chars "$revised_value")
      echo "Revised Value: $revised_value"
      return 0
    elif [[ $REPLY =~ ^[Nn]$ ]]; then
      revised_value=$original_value
      return 1
    else
      echo "Invalid input. Please enter 'y' for yes or 'n' for no."
    fi
  done
}

# Prompt user for accession number
read -p "Please enter the accession number (YYYY-####): " accession

# Prompt user for artist's last name
read -p "Please enter the artist's last name: " artist_last

# Prompt user for artist's first name
read -p "Please enter the artist's first name: " artist_first

# Prompt user for artist's middle name (optional)
read -p "Please enter the artist's middle name (optional): " artist_middle

# Prompt user for title of artwork
read -p "Please enter the title of the artwork: " title

# Replace illegal characters in the inputs
accession=$(replace_illegal_chars "$accession")
artist_last=$(replace_illegal_chars "$artist_last")
artist_first=$(replace_illegal_chars "$artist_first")
artist_middle=$(replace_illegal_chars "$artist_middle")
title=$(replace_illegal_chars "$title")

# Display the entered values for approval
echo "Entered Values:"
echo "Accession Number: $accession"
echo "Artist's Last Name: $artist_last"
echo "Artist's First Name: $artist_first"
echo "Artist's Middle Name: $artist_middle"
echo "Artwork Title: $title"

# Prompt for approval
prompt_revision "Do you approve the entered values?" "y"

if [[ $? -eq 0 ]]; then
  # Display final approval message
  echo "Accession number, artist's names, and artwork title approved."
else
  # Prompt for revision if approval is not given
  prompt_revision "Do you want to revise the accession number?" "$accession"
  accession=$revised_value

  prompt_revision "Do you want to revise the artist's last name?" "$artist_last"
  artist_last=$revised_value

  prompt_revision "Do you want to revise the artist's first name?" "$artist_first"
  artist_first=$revised_value

  prompt_revision "Do you want to revise the artist's middle name?" "$artist_middle"
  artist_middle=$revised_value

  prompt_revision "Do you want to revise the artwork title?" "$title"
  title=$revised_value

  # Display final approval message after revision
  echo "Revised Accession number, artist's names, and artwork title approved."
fi


# Replace spaces with underscores
accession="${accession// /_}"
artist_last="${artist_last// /_}"
artist_first="${artist_first// /_}"
artist_middle="${artist_middle// /_}"

# Remove spaces from the title
title=$(echo "$title" | tr -d ' ')

# Prompt user to drag the existing artwork folder into terminal
cowsay "Please drag the existing ARTWORK FOLDER into the terminal and press Enter. If it doesn't exist, please create the folder now and drag it into the terminal and press Enter."
read -e -p "Artwork folder directory: " artwork_folder

#must contain TBM_Cons

# Create a subdirectory with the accession number
subdirectory="${accession}_${title}"
mkdir -p "${artwork_folder}/${subdirectory}"

# Create subdirectories in the artwork folder's subdirectory
subdirectories=("Acquisition and Registration" "Artist Interaction" "Conservation" "Installation Reports and Exhibition Info" "Photo and Video Documentation" "Research" "Technical Info and Specs")

for subdir in "${subdirectories[@]}"; do
  mkdir -p "${artwork_folder}/${subdirectory}/${subdir}"
  echo "Created subdirectory: ${artwork_folder}/${subdirectory}/${subdir}"
done

# Create additional subdirectories within the Conservation subdirectory
conservation_subdirectories=("Condition Reports" "Equipment Reports")

for subdir in "${conservation_subdirectories[@]}"; do
  mkdir -p "${artwork_folder}/${subdirectory}/Conservation/${subdir}"
  echo "Created subdirectory: ${artwork_folder}/${subdirectory}/Conservation/${subdir}"
done

# Copy the template file to Acquisition and Registration subdirectory
template_file="/Volumes/groups/Conservation/TBM_Cons/DOCUMENTATION TEMPLATES/CataloguingWorksheet_CITI.xlsx"
new_file="${accession}_${artist_last}_CataloguingWorksheet_CITI.xlsx"
cp "$template_file" "${artwork_folder}/${subdirectory}/Acquisition and Registration/${new_file}"
echo "Copied template file to: ${artwork_folder}/${subdirectory}/Acquisition and Registration/${new_file}"

# Copy the template file to Conservation subdirectory
template_file="/Volumes/groups/Conservation/TBM_Cons/DOCUMENTATION TEMPLATES/ActivityLog_Template.docx"
new_file="${accession}_${artist_last}_ActivityLog.docx"
cp "$template_file" "${artwork_folder}/${subdirectory}/Conservation/${new_file}"
echo "Copied template file to: ${artwork_folder}/${subdirectory}/Conservation/${new_file}"

# Confirmation message
cowsay "Subdirectories created and template files copied successfully!"

# Prompt user to drag and drop the directory
cowsay "Please drag and drop the ARTIST-PROVIDED DRIVE into the terminal and press Enter."
read -e -p "Artist Drive path: " artist_drive

# Display message with cowsay
cowsay "Please drag and drop the ARTIST-PROVIDED DRIVE into the terminal and press Enter."

# Display message with cowsay
cowsay "Please drag and drop the DIGITAL REPOSITORY folder into the terminal and press Enter. If it doesn't exist, please create the folder now and drag it into the terminal and press Enter."

# Read dragged digital repository path
read -e -p "Digital Repository path: " digital_repository

# CHANGE THE FOLLOWING TO MAKE THE ALIAS OF THE DIGITAL REPOSITORY
# Extract the directory name from the path
alias=$(basename "$digital_repository")

# Create a symbolic link (alias) to the directory inside the artwork_folder
ln -s "$digital_repository" "$artwork_folder/$subdirectory/$alias"

echo "Alias created for $digital_repository in $artwork_folder."

# Generate tree command output and save it to a file
tree_output_file="${artwork_folder}/${subdirectory}/Conservation/Condition Reports/"${accession}"-ARC-1_tree.txt"
tree "$artist_drive" > "$tree_output_file"

echo "Tree command output for the source directory saved to: $tree_output_path"

# Log file path with accession number and artist last name
log_file_path="${artwork_folder}/${subdirectory}/Conservation/Condition Reports/"${accession}"-ARC-1_ingest.log"

# Perform the file ingestion without transferring hidden files using rsync with progress and logging
rsync -aihW --progress --log-file="$log_file_path" "$artist_drive" "$digital_repository" ##if there is an issue remove --exclude '.*' from this line


echo "Files successfully ingested using rsync!"

# Capture video files in the digital repository directory and store them in an array
video_files=()
while IFS= read -r -d '' file; do
  video_files+=("$file")
done < <(find "$digital_repository" -type f \( -iname "*.mp4" -o -iname "*.mov" -o -iname "*.avi" \) -print0)

# Print the number of video files found and list them (for debugging)
echo "Found ${#video_files[@]} video files in the digital repository directory."
printf '%s\n' "${video_files[@]}"

# Process each video file in the digital repository directory
for file in "${video_files[@]}"; do
  # Extract the file name and extension
  file_basename=$(basename "$file")
  file_name="${file_basename%.*}"
  file_extension="${file_basename##*.}"

  # Calculate the MD5 checksum of the digital repository file
  digital_repository_checksum=$(md5 -r "$file" | awk '{ print $1 }') 
  # Create the checksum file with the desired format
  checksum_file="${artwork_folder}/${subdirectory}/Conservation/Condition Reports/${file_name}-md5.txt"
  echo "MD5 ${file_basename} = ${digital_repository_checksum}" > "${checksum_file}"
  echo "Checksum saved to: ${checksum_file}"

  # Capture mediainfo -f output
  mediainfo_output="${artwork_folder}/${subdirectory}/Conservation/Condition Reports/${file_name}-mediainfo.txt"
  mediainfo -f "$file" > "$mediainfo_output"
  echo "mediainfo captured and saved to: ${mediainfo_output}"

  # Capture ffmpeg framemd5 output
  framemd5_output="${artwork_folder}/${subdirectory}/Conservation/Condition Reports/${file_name}.framemd5"
  echo "ffmpeg framemd5 for ${file_basename}" > "$framemd5_output"  # Add the header
  ffmpeg -i "$file" -f framemd5 - >> "$framemd5_output" 2>/dev/null  # Append the framemd5 output
  echo "ffmpeg framemd5 captured and saved to: ${framemd5_output}"

  # Checksum file on the artist's drive
  artist_file="${artist_drive}/${file_basename}"
  
  # Calculate the MD5 checksum of the original file in the artist_drive
  artist_checksum=$(md5 -r "$artist_file" | awk '{ print $1 }')

   # Compare checksums
  if [[ "${digital_repository_checksum}" == "${artist_checksum}" ]]; then
    cowsay "Checksum verification passed for ${file_basename}"
  else
    cowsay "Checksum verification failed for ${file_basename}
Digital Repository Checksum: ${digital_repository_checksum}
Artist Drive Checksum: ${artist_checksum}"
  fi
done

  # DO NOT CHANGE ABOVE THIS LINE



