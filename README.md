# AI-Boosted-Intrusion-Sniffer
AI-Boosted Intrusion Sniffer

Hey, I’m Raj—a cybersecurity nut with a Master’s in Info Systems and Security. This project amps up my old Intrusion Detection System (IDS) game by tossing AI into the mix. I used to run Snort to catch network threats, but now I’ve got AI sniffing out weird patterns Snort might miss. It’s like giving my guard dog a superpower!

What’s the Big Idea?

Networks get hit with sneaky attacks—like someone probing your ports or flooding your Wi-Fi. Snort catches the obvious stuff, but AI digs deeper, learning what’s “normal” for my network and flagging the oddballs. Think of it as a smarter IDS.

How I Made It Happen

Here’s the rundown, like I’m walking you through it:

1. Capture Traffic: I fired up Wireshark to grab network chatter—like “192.168.1.10 hitting port 80 a bunch.” Saved it as a .pcap file.
   Example: My laptop streaming YouTube vs. some random IP scanning me.

2. Crunch the Data: Turned that traffic into numbers—packet sizes, times, IPs. Like “10 packets, 500 bytes, 2 seconds apart.”
   Macro Detail: Used “pandas” to organize it—like a spreadsheet for nerds.

3. Train the AI: Fed it a day’s worth of my normal traffic (Netflix, emails), then taught it to spot weirdness—like 1000 tiny packets in a minute.
   Example: Normal: 50 packets to Google. Weird: 2000 to some unknown IP.
   Macro Detail: Went with “scikit-learn’s Isolation Forest”—it’s killer at finding outliers.

4. Catch Intruders: Ran it live—AI flags anything funky, like a port scan I simulated with Nmap.
   Example: “Alert! 192.168.1.50 looks sus—too many packets!”

Stuff You’ll Need

Python: Grab it from python.org—my main tool.
Helpers:
- scikit-learn—for the AI smarts.
- pandas—to handle data.
- pyshark—to read Wireshark files in Python.
Wireshark: Free network sniffer—love this thing.
Your Wi-Fi: Test on your own network.

Let’s Build It—Step by Step with Commands

1. Install Python: Hit python.org, download 3.11, install with “Add to PATH” checked.
   Command: In command line (Windows: “cmd”; Mac: “terminal”), type “python --version”. Should say “Python 3.11.x”.

2. Get Helpers: In command line, run these one by one:
   - “pip install scikit-learn” (AI brain).
   - “pip install pandas” (data organizer).
   - “pip install pyshark” (traffic reader—might need Wireshark installed first).

3. Set Up Folder: Make “IntrusionSniffer” on your desktop.
   Command: “cd Desktop\IntrusionSniffer” (Windows) or “cd ~/Desktop/IntrusionSniffer” (Mac).

4. Grab Some Traffic: Open Wireshark, pick your Wi-Fi, hit “Start.” Surf normally for 5 mins, stop, save as “normal.pcap” in your folder.

5. Code It: Open Notepad, paste this, save as “intrusion_sniffer.py”:
```python
# My AI intrusion catcher
import pyshark
import pandas as pd
from sklearn.ensemble import IsolationForest

# Read traffic
cap = pyshark.FileCapture("normal.pcap")
data = []
for pkt in cap:
    try:
        data.append([pkt.length, pkt.ip.src, pkt.ip.dst])
    except AttributeError:
        pass

# Make it numbers
df = pd.DataFrame(data, columns=["size", "src", "dst"])
df["size"] = df["size"].astype(float)

# Train AI
model = IsolationForest(contamination=0.1)  # 10% might be weird
model.fit(df[["size"]])  # Just size for now—add more later

# Test it
test_data = [[100], [2000], [50]]  # Fake new packets
preds = model.predict(test_data)
for i, pred in enumerate(preds):
    if pred == -1:
        print(f"Alert! Packet {i} looks sus—size {test_data[i][0]}!")
    else:
        print(f"Packet {i} is chill.")
Run It: In command line (after “cd” to IntrusionSniffer), type “python intrusion_sniffer.py”. See what it flags!
Bumps in the Road

Small Data: 2 minutes of traffic sucks—grab at least 5-10 mins.
Tuning: Too many alerts? Tweak “contamination” (like 0.05).
Windows Hiccups: Pyshark might whine—install Wireshark first.

Why I Love It

Combines my Snort skills with AI—catches stuff manually I’d miss. Perfect for my auditing vibe.
