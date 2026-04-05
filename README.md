AI-Boosted-Intrusion-Sniffer:
Signature-based intrusion detection tools are reliable for known threats. The problem is that attackers know this too. Novel attack patterns, slow reconnaissance, and low-volume probing can slip past rule-based systems entirely because they do not match any existing signature.

This project tackles that gap using anomaly detection — training a model on what normal network traffic looks like, then flagging anything that deviates from it.


Why I Built This
Working with network monitoring tools taught me that the most dangerous traffic is often the quietest. A port scan spread over hours looks nothing like the aggressive scans Snort rules are written to catch. Anomaly-based detection catches those slow, deliberate probes that signature tools miss — and that is exactly what this project demonstrates.



What It Does
Captures network traffic using Wireshark, parses the packet capture with pyshark, trains an Isolation Forest model on normal traffic patterns, and flags packets or sessions that deviate significantly from that baseline as potential intrusions.


Tools and Libraries
Python 3.11 — core language
Wireshark — packet capture
pyshark — Python interface for reading pcap files
scikit-learn — Isolation Forest anomaly detector
pandas — data structuring and feature engineering



How It Works

Step 1 — Capture Baseline Traffic
Open Wireshark, select your active network interface, and capture 10+ minutes of normal activity — browsing, emails, streaming. Save as normal.pcap in your project folder.
The more baseline data you capture the better the model understands what normal looks like for your specific environment.

Step 2 — Install Dependencies
pip install scikit-learn
pip install pandas
pip install pyshark
Note: pyshark requires Wireshark to be installed first as it depends on the tshark binary.


Step 3 — Train and Detect
Save as intrusion_sniffer.py:
import pyshark
import pandas as pd
from sklearn.ensemble import IsolationForest

# Parse packet capture
cap = pyshark.FileCapture("normal.pcap")
data = []

for pkt in cap:
    try:
        data.append({
            "size": float(pkt.length),
            "src": pkt.ip.src,
            "dst": pkt.ip.dst
        })
    except AttributeError:
        pass  # Skip non-IP packets

# Build feature dataframe
df = pd.DataFrame(data)

# Train anomaly detection model
# contamination = expected proportion of anomalies in baseline
model = IsolationForest(contamination=0.05, random_state=42)
model.fit(df[["size"]])

# Evaluate new incoming packets
new_packets = pd.DataFrame([[100], [2000], [50], [5000]], columns=["size"])
predictions = model.predict(new_packets)

print("\n=== INTRUSION SNIFFER RESULTS ===\n")
for i, pred in enumerate(predictions):
    size = new_packets["size"][i]
    if pred == -1:
        print(f"  [ALERT]  Packet {i+1} — size {size} bytes — anomalous pattern detected")
    else:
        print(f"  [CLEAR]  Packet {i+1} — size {size} bytes — within normal range")
Run it:
python intrusion_sniffer.py


Step 4 — Sample Output
=== INTRUSION SNIFFER RESULTS ===

  [CLEAR]  Packet 1 — size 100 bytes  — within normal range
  [ALERT]  Packet 2 — size 2000 bytes — anomalous pattern detected
  [CLEAR]  Packet 3 — size 50 bytes   — within normal range
  [ALERT]  Packet 4 — size 5000 bytes — anomalous pattern detected
Tuning the Model
Too many false alerts — lower the contamination value:
IsolationForest(contamination=0.02)  # More conservative
Missing real threats — raise it slightly:
IsolationForest(contamination=0.10)  # More sensitive
Add more features for better accuracy — packet timing, port numbers, protocol type, and session duration all significantly improve detection quality beyond packet size alone.




Limitations and Next Steps
Single feature model — packet size alone is not enough for production detection. Time intervals, destination port distribution, and IP reputation scoring would make this significantly more robust.

Offline analysis — this version runs on saved pcap files. Real-time detection would require streaming the pyshark capture directly into the model pipeline.

Environment specific — the baseline must match the environment being monitored. A model trained on home network traffic will generate excessive alerts on enterprise traffic and vice versa.



What I Learned:
How anomaly detection differs fundamentally from signature-based detection and why both are needed
Why baselining normal behavior is as important as knowing what attacks look like
How Isolation Forest identifies outliers by measuring how easily a data point can be separated from the rest
The practical challenges of tuning detection sensitivity in a real network environment.


$ echo connect_with_me:
╔═════════════════════════════════════╗
║  LinkedIn  →  linkedin.com/in rajesh-rathlavathu23  ║
║  Portfolio →  Richierich69696.github.io              ║
║  Email     →  rajeshrathlavathu@gmail.com            ║
║  GitHub    →  github.com/Richierich69696              ║
╚═════════════════════════════════════╝