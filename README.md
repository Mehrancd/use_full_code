**usefull code in medical image processing and cluster usage**


*-how to find a file in current folder and subfolders in cluster:*


     $find ./ -type f -name "*part_of_file*"$

*-How to automatically set lung window for CT images in Slicer (the parameters inside window level can be modified as you require):*

in slicer python interactor window:


    all_nodes = slicer.mrmlScene.GetNodesByClass('vtkMRMLScalarVolumeNode')
    for ct in all_nodes:
        if isinstance(ct, slicer.vtkMRMLLabelMapVolumeNode):
            continue
        else:
            ct.GetDisplayNode().SetAutoWindowLevel(False)
            ct.GetDisplayNode().SetWindowLevel(1500,-500) 
     

*-How to stop repeating password on ssh cluster:*

    $ssh-keygen -t rsa -b 4096 -C "your_email@ucl.ac.uk"
    $ssh-copy-id -i ~/.ssh/id_rsa.pub cluster_ID@comic
  
 three times Enter->Enter->Enter
 
 then each time you only need to do:
 
    $ ssh cluster_ID@comic

*-How to pull files from cluster to your local machine:*

    $method 1: rsync
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

*-how to hide current running code in terminal:*

    $install screen: sudo apt install screen
hide current processing: 

    $screen
bring back current procesing:

    $screen -r
    
*-how to find a file in directory or subdirectory in terminal:*

    $find /path_to_directory/ -type f -name "COPDGene_A68731*"

*-How to generate 3D rendered label for QC in Slicer:*

in slicer python interactor window:


     import csv
     import DICOMScalarVolumePlugin
     import os
     import pandas as pd
     input_path1='/media/mehran/SSD Storage 2/AAVS/COPDGene_AAVS'
     output_path='/media/mehran/SSD Storage 2/AAVS/COPDGene_AAVS_2d'
     list_files_xlsx=pd.read_excel('/media/mehran/SSD Storage 2/AAVS/List_QC_COPDGene_half.xlsx')
     print('number of summit subjects:',len(list_files_xlsx))
     list_files=[f for f in os.listdir(input_path1) if f.endswith('.nii.gz')]
     for i in range(len(list_files_xlsx)):
         ID=list_files_xlsx.at[i,'ID']
         if ID+'.nii.gz' in list_files:
             slicer.util.loadLabelVolume(input_path1+'/'+ID+'.nii.gz')
             labelMapNode = getNode(ID)
             thresholdValue = 1
             segmentationNode = slicer.mrmlScene.AddNewNodeByClass("vtkMRMLSegmentationNode",'Segm1')
             slicer.modules.segmentations.logic().ImportLabelmapToSegmentationNode(labelMapNode, segmentationNode)
             segmentIDs = vtk.vtkStringArray()
             segmentationNode.GetSegmentation().GetSegmentIDs(segmentIDs)
             displayNode = slicer.mrmlScene.AddNewNodeByClass("vtkMRMLSegmentationDisplayNode",'disp_segm')
             segmentationNode.SetAndObserveDisplayNodeID(displayNode.GetID())
             segmentationDisplayNode = segmentationNode.GetDisplayNode()
             segmentationDisplayNode.SetSegmentVisibility(segmentIDs.GetValue(0), False)
             segmentationDisplayNode.SetSegmentVisibility(segmentIDs.GetValue(2), False)
             slicer.modules.segmentations.logic().ExportVisibleSegmentsToModels(segmentationNode, 0)
             segmentationDisplayNode.SetVisibility3D(True)
             view = slicer.app.layoutManager().threeDWidget(0).threeDView()
             view.resetFocalPoint()
             view.mrmlViewNode().SetBackgroundColor(0,0,0)
             view.mrmlViewNode().SetBackgroundColor2(0,0,0)
             view.mrmlViewNode().SetBoxVisible(False)
             view.mrmlViewNode().SetAxisLabelsVisible(False)
             view.resetCamera()
             view.forceRender()
             renderWindow = view.renderWindow()
             renderWindow.SetAlphaBitPlanes(1)
             wti = vtk.vtkWindowToImageFilter()
             wti.SetInputBufferTypeToRGBA()
             wti.SetInput(renderWindow)
             writer = vtk.vtkJPEGWriter()
             writer.SetFileName(output_path+'/'+ID+'.jpg')
             writer.SetInputConnection(wti.GetOutputPort())
             writer.Write()
             slicer.mrmlScene.Clear(0)
              >>> 

*-how to estimate thoracic volume:*

     for coronal_idx in range(lung_mask.shape[1]):
         coronal_slice = lung_mask[:, coronal_idx, :]    
         for row_idx in range(coronal_slice.shape[0]):
             row = coronal_slice[row_idx, :]        
             non_zero_indices = np.nonzero(row)[0]        
             if non_zero_indices.size > 0:
                 first_idx = non_zero_indices[0]
                 last_idx = non_zero_indices[-1]            
                 row[first_idx:last_idx+1] = 1       
             coronal_slice[row_idx, :] = row    
         lung_mask[:, coronal_idx, :] = coronal_slice
     filled_lung_mask_sitk = sitk.GetImageFromArray(lung_mask)
     filled_lung_mask_sitk.CopyInformation(img)
     sitk.WriteImage(filled_lung_mask_sitk, 'filled_lung_mask.nii.gz')

     
*-how to combine a folder of xlsx files into one:*

     import pandas as pd
     import glob
     import argparse
     import os
     import numpy as np
     
     def merge_excel_files(input_folder, output_file):
         # Construct the file path pattern for all .xlsx files in the input folder
         file_paths = glob.glob(os.path.join(input_folder, "*.xlsx"))
         
         dataframes = []
         for file_path in file_paths:
             df = pd.read_excel(file_path)
             df = df.replace(0, np.nan)
             dataframes.append(df)
         
         # Merge DataFrames by taking the first occurrence of each ID
         merged_df = pd.concat(dataframes, axis=0, join='outer', ignore_index=True).groupby('ID', as_index=False).first()
         
         # Remove any duplicate rows based on the 'ID' column
         merged_df = merged_df.drop_duplicates(subset=['ID'])
         
         # Save the merged DataFrame to the specified output Excel file
         merged_df.to_excel(output_file, index=False)
         print(f"Data merged and saved to {output_file}")
     
     if __name__ == "__main__":
         # Set up argument parsing
         parser = argparse.ArgumentParser(description="Merge .xlsx files in a folder and save the result.")
         parser.add_argument("input_folder", type=str, help="Path to the folder containing .xlsx files to merge")
         parser.add_argument("output_file", type=str, help="Path to save the merged output Excel file")
         
         # Parse the command-line arguments
         args = parser.parse_args()
         
         # Call the merge function with provided arguments
         merge_excel_files(args.input_folder, args.output_file)
