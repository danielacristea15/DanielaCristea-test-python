# DanielaCristea-test-python

import os
import sys
import shutil
import time
import hashlib

def calculate_md5(file_path):
    """Calculate the MD5 hash of a file."""
    hash_md5 = hashlib.md5()
    with open(file_path, "rb") as f:
        for chunk in iter(lambda: f.read(4096), b""):
            hash_md5.update(chunk)
    return hash_md5.hexdigest()

def synchronize_folders(source_folder, replica_folder, log_file_path):
    """Synchronize the content of the source folder to the replica folder."""
    try:
        with open(log_file_path, "a") as log_file:
            for root, _, files in os.walk(source_folder):
                for file in files:
                    source_file_path = os.path.join(root, file)
                    replica_file_path = source_file_path.replace(source_folder, replica_folder)
                    
                    # Check if file needs to be updated or created
                    if not os.path.exists(replica_file_path) or calculate_md5(source_file_path) != calculate_md5(replica_file_path):
                        shutil.copy2(source_file_path, replica_file_path)
                        log_file.write(f"Copied: {source_file_path} to {replica_file_path}\n")
            
            # Remove files in replica folder not present in source folder
            for root, _, files in os.walk(replica_folder):
                for file in files:
                    replica_file_path = os.path.join(root, file)
                    source_file_path = replica_file_path.replace(replica_folder, source_folder)
                    
                    if not os.path.exists(source_file_path):
                        os.remove(replica_file_path)
                        log_file.write(f"Removed: {replica_file_path}\n")
    except Exception as e:
        print(f"An error occurred: {str(e)}")

if __name__ == "__main__":
    if len(sys.argv) != 4:
        print("Usage: python sync_folders.py <source_folder> <replica_folder> <log_file_path>")
        sys.exit(1)
    
    source_folder = sys.argv[1]
    replica_folder = sys.argv[2]
    log_file_path = sys.argv[3]
    
    # Create replica folder if it doesn't exist
    if not os.path.exists(replica_folder):
        os.makedirs(replica_folder)
    
    while True:
        synchronize_folders(source_folder, replica_folder, log_file_path)
        print("Synchronization completed.")
        time.sleep(3600)  # Synchronize every hour (3600 seconds)
