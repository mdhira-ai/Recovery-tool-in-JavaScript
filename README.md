## *creating a basic recovery tool in JavaScript:*

 - Install the `node-disk-manager` library for low-level disk access:
> npm install node-disk-manager

    const diskManager = require('node-disk-manager');
    const fs = require('fs');

	const diskPath = "/dev/sda1"; // Replace with the correct path
	
	// Scan for Deleted Files
	function scanForDeletedFiles(diskPath) {
	    return new Promise((resolve, reject) => {
	        diskManager.open(diskPath, (err, disk) => {
	            if (err) {
	                reject(err);
	            } else {
	                disk.scan((err, files) => {
	                    if (err) {
	                        reject(err);
	                    } else {
	                        resolve(files);
	                    }
	                });
	            }
	        });
	    });
	}

	// Recover a Specific File:

	function recoverFile(diskPath, fileInfo) {
	    return new Promise((resolve, reject) => {
	        diskManager.open(diskPath, (err, disk) => {
	            if (err) {
	                reject(err);
	            } else {
	                disk.readBlock(fileInfo.offset, fileInfo.size, (err, data) => {
	                    if (err) {
	                        reject(err);
	                    } else {
	                        fs.writeFile(`recovered_files/${fileInfo.filename}`, data, (err) => {
	                            if (err) {
	                                reject(err);
	                            } else {
	                                resolve(fileInfo.filename);
	                            }
	                        });
	                    }
	                });
	            }
	        });
	    });
	}

	(async () => {
	    try {
	        const deletedFiles = await scanForDeletedFiles(diskPath);

	        if (deletedFiles.length > 0) {
	            fs.mkdirSync('recovered_files'); // Create output directory

	            for (const fileInfo of deletedFiles) {
	                const recoveredFilename = await recoverFile(diskPath, fileInfo);
	                console.log(`Recovered file: ${recoveredFilename}`);
	            }
	        } else {
	            console.log("No deleted files found.");
	        }
	    } catch (error) {
	        console.error("Error:", error);
	    }
	})();





