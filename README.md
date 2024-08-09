usefull code in medical image processing and cluster usage


-how to find a file in current folder and subfolders in cluster:


     $find ./ -type f -name "*part_of_file*"$

-How to automatically set lung window for CT images in Slicer (the parameters inside window level can be modified as you require):

in slicer python interactor window:


    all_nodes = slicer.mrmlScene.GetNodesByClass('vtkMRMLScalarVolumeNode')
    for ct in all_nodes:
        ct.GetDisplayNode().SetAutoWindowLevel(False)
        ct.GetDisplayNode().SetWindowLevel(1500,-500) 
    >>> 

How to stop repeating password on ssh cluster:

    $ssh-keygen -t rsa -b 4096 -C "your_email@ucl.ac.uk"
    $ssh-copy-id -i ~/.ssh/id_rsa.pub cluster_ID@comic
  
 three times Enter->Enter->Enter
 
 then each time you only need to do:
 
    $ ssh cluster_ID@comic

- How to pull files from cluster to your local machine:
    method 1: rsync
    method 2: in linux files→ "Other Locations" → Enter server addresses : ssh://cluster_ID@... and copy-paste
    method 3: create list files (e.g., remote_listing.txt) and run a bash file:
    REMOTE_USER="youruser"
    REMOTE_HOST="comic" # Use little as it is the most stable cluster node
    REMOTE_PATH="/cluster/project.../"
    LOCAL_PATH="/path_to_local/"
    RETRY_LIMIT=10
    SLEEP_TIME=5
    FILE_LIST="remote_listing.txt" # Text file containing the list of files to be transferred
    # Function to perform scp
    perform_scp() {
    local file=$1
    scp "$REMOTE_USER@$REMOTE_HOST:$REMOTE_PATH/$file" "$LOCAL_PATH"
    }
    # Main loop to iterate over files and retry if necessary
    while IFS= read -r file; do
    # Check if file already exists locally
    if [[ -f "$LOCAL_PATH/$file" ]]; then
    echo "File $file already exists locally. Skipping transfer."
    continue
    fi
     RETRY_COUNT=0
    while (( RETRY_COUNT < RETRY_LIMIT )); do
    perform_scp "$file"
     # Check the exit status of scp
    if [[ $? -eq 0 ]]; then
    echo "Successfully transferred $file."
    break
    else
    (( RETRY_COUNT++ ))
    echo "Failed to transfer $file. Attempt $RETRY_COUNT of $RETRY_LIMIT. Retrying in $SLEEP_TIME seconds..."
    sleep $SLEEP_TIME
    fi
    done
     if (( RETRY_COUNT == RETRY_LIMIT )); then
    echo "Failed to transfer $file after $RETRY_LIMIT attempts."
    fi
    done < "$FILE_LIST"
    echo "File transfer process completed."

- How to hide current running code in terminal:
1- install screen: sudo apt install screen
 hide current processing: 
   $ screen
 bring back current procesing:
   $ screen -r
