# 2b IMPLEMENTATION OF SLIDING WINDOW PROTOCOL
## AIM
## ALGORITHM:
1. Start the program.
2. Get the frame size from the user
3. To create the frame based on the user request.
4. To send frames to server from the client side.
5. If your frames reach the server it will send ACK signal to client
6. Stop the Program
## PROGRAM

import time
import random

class Packet:

    def __init__(self, data, seq_no):
        self.data = data
        self.seq_no = seq_no

    def __str__(self):
        return f"Packet(Data: '{self.data}', Seq: {self.seq_no})"

class Sender:

    def __init__(self, receiver, window_size):
        self.receiver = receiver
        self.window_size = window_size
        self.next_seq_no = 0
        self.base = 0  # Starting sequence number of the current window
        self.packets_in_flight = {}  # Store unacknowledged packets (seq_no: packet)

    def send(self, data_list):
        num_packets = len(data_list)
        while self.base < num_packets:
            # Send packets within the current window
            while self.next_seq_no < self.base + self.window_size and self.next_seq_no < num_packets:
                packet = Packet(data_list[self.next_seq_no], self.next_seq_no)
                print(f"Sender: Sending packet {packet}")
                self.packets_in_flight[self.next_seq_no] = packet
                self.receiver.receive(packet)
                self.next_seq_no += 1

            # Wait for acknowledgements
            ack = self.wait_for_ack()
            if ack is not None:
                # Move the window based on the received ACK
                while self.base <= ack:
                    if self.base in self.packets_in_flight:
                        del self.packets_in_flight[self.base]
                    self.base += 1
                print(f"Sender: Received ACK {ack}. Window moved to {self.base}")
            else:
                # Timeout: Resend all unacknowledged packets in the current window
                print("Sender: Timeout! Resending unacknowledged packets...")
                for seq_no, packet in self.packets_in_flight.items():
                    print(f"Sender: Resending packet {packet}")
                    self.receiver.receive(packet)

            time.sleep(0.5) # Introduce a small delay

    def wait_for_ack(self, timeout=2):
        """Simulates waiting for an ACK with a timeout."""
        time.sleep(timeout * random.uniform(0.8, 1.2)) # Simulate network delay
        return self.receiver.acknowledgement

class Receiver:

    def __init__(self, window_size):
        self.window_size = window_size
        self.expected_seq_no = 0
        self.buffer = {}  # Store received packets out of order
        self.acknowledgement = None

    def receive(self, packet):
        print(f"Receiver: Received packet {packet}")
        seq_no = packet.seq_no

        if self.expected_seq_no <= seq_no < self.expected_seq_no + self.window_size:
            if seq_no not in self.buffer:
                self.buffer[seq_no] = packet
                print(f"Receiver: Packet {seq_no} buffered.")

            # Check if we can deliver packets in order
            while self.expected_seq_no in self.buffer:
                delivered_packet = self.buffer.pop(self.expected_seq_no)
                print(f"Receiver: Delivering packet {delivered_packet}")
                self.expected_seq_no += 1

            # Send ACK for the highest correctly received packet in order
            self.acknowledgement = self.expected_seq_no - 1
            print(f"Receiver: Sending ACK {self.acknowledgement}")

        elif seq_no < self.expected_seq_no:
            # Resend ACK for previously received packet (for duplicates)
            print(f"Receiver: Duplicate packet {packet}. Resending ACK {seq_no}")
            self.acknowledgement = seq_no

        else:
            print(f"Receiver: Packet {seq_no} out of window. Discarding.")
            
if __name__ == "__main__":

    window_size = 4
    receiver = Receiver(window_size)
    sender = Sender(receiver, window_size)

    data_to_send = [f"Data {i}" for i in range(10)]

    print(f"\n--- Starting Sliding Window Protocol with Window Size: {window_size} ---")
    sender.send(data_to_send)

    print("\n--- Transmission complete ---")
## OUPUT

![image](https://github.com/user-attachments/assets/67da7a7e-ca7b-4a6e-9168-0d322593a5ce)

## RESULT
Thus, python program to perform stop and wait protocol was successfully executed
