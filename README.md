const arp = require('node-arp');
const fs = require('fs');

const ipAddresses = [
  '192.168.1.1', // Replace with actual IP addresses you want to check
  '192.168.1.2',
  '192.168.1.3'
];

function scanNetwork() {
  const arpEntries = [];

  function scanIp(ip, callback) {
    arp.getMAC(ip, (err, mac) => {
      if (err) {
        console.log(`Error retrieving MAC for IP: ${ip}`);
      } else if (mac) {
        console.log(`IP: ${ip}, MAC: ${mac}`);
        arpEntries.push({ ip, mac });
      } else {
        console.log(`No ARP entry found for IP: ${ip}`);
      }
      callback();
    });
  }

  function scanAllIps(index) {
    if (index >= ipAddresses.length) {
      // Save the ARP entries to a JSON file
      fs.writeFile('arp-entries.json', JSON.stringify(arpEntries, null, 2), (err) => {
        if (err) {
          console.log('Error saving ARP entries:', err);
        } else {
          console.log('ARP entries saved to arp-entries.json');
        }
      });
      return;
    }

    // Scan the current IP and move to the next one
    scanIp(ipAddresses[index], () => {
      scanAllIps(index + 1);
    });
  }

  // Start the scan
  scanAllIps(0);
}

// Function to reset the ARP list every 15 minutes
function startTimer() {
  scanNetwork();

  // Set a timer for 15 minutes (15 * 60 * 1000 milliseconds)
  setInterval(() => {
    console.log('\nRescanning network and resetting the ARP list...');
    scanNetwork();
  }, 15 * 60 * 1000);
}

// Start the initial scan and timer
startTimer();
