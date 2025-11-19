# voting machine

## real-World Context for Voting Machine

Electronic Voting Machines (EVMs) play a crucial role in modern democratic systems by ensuring fair, accurate, and transparent elections. Traditional paper-based voting methods often suffer from challenges such as manual counting errors, time delays, ballot tampering, and high operational costs. With increasing population and large-scale elections, it becomes essential to adopt systems that are fast, reliable, and less prone to human error.

An electronic voting system overcomes these limitations by providing:

High counting accuracy: Electronic counters minimize mistakes that commonly occur during manual vote counting.

Faster results: Votes are recorded and stored instantly, allowing results to be generated within seconds after the election ends.

Improved security: Memory modules prevent tampering and ensure that each vote is counted only once.

User-friendly operation: Simple push-button interfaces make the system accessible to all voters.

Data integrity: Using ROM for candidate data and RAM for vote counts enables secure storage and controlled access.

In real-world elections—such as college elections, local body elections, corporate voting, and public decision-making—electronic voting machines greatly enhance trust and transparency. They ensure that every vote is recorded accurately and every candidate receives the correct number of votes without manual interference.

Thus, electronic voting systems contribute significantly to fair elections, strong democracy, and transparent decision-making, making them essential in today’s fast-growing digital world.

## System Architecture

The Electronic Voting Machine (EVM) is designed using a modular and hierarchical architecture to ensure accuracy, reliability, and secure handling of voter inputs. The system is divided into four major functional blocks—Input Unit, Counting Module, Memory Model, and Display Module—coordinated by a central Control FSM (Finite State Machine). Each block performs a dedicated operation and communicates with other modules through well-defined signals.

## block diagram
<img width="1445" height="546" alt="Screenshot 2025-11-19 090751" src="https://github.com/user-attachments/assets/6f9444fa-3f44-42d6-b1ca-5ad85a3c97de" />

## Design Morphology

The design morphology of the Electronic Voting Machine (EVM) is based on four major functional modules. Each module performs a specific role in the voting process and interacts with the others to achieve accurate and secure vote recording and result display.

1. Input Unit – Push-Button Based Voting
Function

To capture a voter’s choice through push-button switches.

Morphology

One push-button is assigned to each candidate.

A debouncing mechanism is used to remove mechanical noise in button presses.

A one-shot pulse generator converts any long or bouncy button press into a single clean pulse.

The output of this module is a vote pulse, which ensures one vote is recorded for each button press.

Output

vote_pulse[i] → Indicates a vote cast for candidate i.

2. Counting Module – Vote Storage and Increment
Function

To store and increment the vote count for each candidate.

Morphology

Contains multiple counters, one for each candidate.

Each vote_pulse[i] from the Input Unit increments the corresponding candidate’s counter.

Counters are enabled only during active voting (controlled by the main FSM).

Uses fixed bit-width registers to prevent overflow.

Output

count[i] → Total votes stored for candidate i.

3. Memory Model – ROM for Candidate Data & RAM for Votes
Function

To store candidate information and maintain vote data.

Morphology
ROM (Read-Only Memory)

Stores static candidate information such as:

Candidate ID

Candidate name

Candidate symbol

Loaded at system initialization and cannot be changed during voting.

RAM / Register File

Stores dynamic vote counts for each candidate.

Allows read/write access by the Counting Module.

Organizes vote data in a secure and isolated manner.

Output

Candidate details from ROM.

Vote count data from RAM/registers.

4. Display Module – Winner and Result Output
Function

To show election results after voting ends.

Morphology

Receives all vote counts from the Counting/Memory modules.

Uses comparison logic to identify the candidate with the highest number of votes.

Displays:

Winner candidate index/ID

Vote counts

Tie indicators (if required)

Can interface with:

7-segment display

LCD

LEDs

Console output (in simulation)

Output

winner_idx → Candidate with the highest votes
count_display → Visual output of each candidate’s vote count
## verilog code 
module voting_machine (
    input clk, reset, vote1, vote2, vote3, end_vote,
    output reg [3:0] count1, count2, count3,
    output reg [1:0] winner
);

    // State encoding
    localparam IDLE = 2'b00, VOTING = 2'b01, RESULT = 2'b10;
    reg [1:0] state;

    always @(posedge clk or posedge reset) begin
        if (reset) begin
            count1 <= 0; count2 <= 0; count3 <= 0;
            winner <= 0; 
            state <= IDLE;
        end else begin
            case (state)
                IDLE: begin
                    if (!end_vote)
                        state <= VOTING;
                end

                VOTING: begin
                    if (vote1) count1 <= count1 + 1;
                    else if (vote2) count2 <= count2 + 1;
                    else if (vote3) count3 <= count3 + 1;

                    if (end_vote) state <= RESULT;
                end

                RESULT: begin
                    if (count1 > count2 && count1 > count3) winner <= 1;
                    else if (count2 > count1 && count2 > count3) winner <= 2;
                    else if (count3 > count1 && count3 > count2) winner <= 3;
                    else winner <= 0; // tie case
                end
            endcase
        end
    end
endmodule







  

## test bench 

module voting_machine_tb;

    reg clk;
    reg rst_n;
    reg start;
    reg stop;
    reg [3:0] vote_btn;

    wire [1:0] winner;
    wire [7:0] vote_count0, vote_count1, vote_count2, vote_count3;

    voting_machine dut(
        .clk(clk),
        .rst_n(rst_n),
        .start(start),
        .stop(stop),
        .vote_btn(vote_btn),
        .winner(winner),
        .vote_count0(vote_count0),
        .vote_count1(vote_count1),
        .vote_count2(vote_count2),
        .vote_count3(vote_count3)
    );

    always #5 clk = ~clk;

    initial begin
        clk = 0;
        rst_n = 0;
        start = 0;
        stop = 0;
        vote_btn = 0;

        #20 rst_n = 1;

        #10 start = 1;
        #10 start = 0;

        #10 vote_btn = 4'b0001;
        #10 vote_btn = 4'b0010;
        #10 vote_btn = 4'b0010;
        #10 vote_btn = 4'b0100;
        #10 vote_btn = 4'b0001;
        #10 vote_btn = 4'b1000;
        #10 vote_btn = 4'b0010;

        #10 vote_btn = 4'b0000;

        #20 stop = 1;
        #10 stop = 0;

        #20;
        $display("================================");
        $display("Candidate0 = %d", vote_count0);
        $display("Candidate1 = %d", vote_count1);
        $display("Candidate2 = %d", vote_count2);
        $display("Candidate3 = %d", vote_count3);
        $display("Winner = %d", winner);
        $display("================================");

        #20 $finish;
    end

endmodule

## output
<img width="1528" height="955" alt="image" src="https://github.com/user-attachments/assets/c822f2d6-190a-401b-aeab-d3daf18629f6" />

 ## result
The Verilog-based electronic voting machine was successfully designed, simulated, and tested. The system accurately recorded votes for each candidate, stored them in internal registers, and correctly identified the winning candidate based on the highest vote count.





