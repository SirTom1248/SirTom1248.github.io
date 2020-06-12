<!DOCTYPE html>
<html>
<body>
    <script>
        function hhConvert() {
            //This combines all the sub functions to convert the actual HH
            document.getElementById('hhOutput').value = "";

            var hhTypeNum = hhType();
            if (hhTypeNum == 1) {
                hhTypeA();
            }
            else {
                document.getElementById('hhOutput').value = "An error has occured. Please try again.";
                return;
            }

        }

        function hhType() {
            //This should detect whether NLHE/PLO/Stud/etc
            var hhText = document.getElementById('hhInput').value;
            var hhFLB = hhText.indexOf("\n");
            var hhFirstLine = hhText.substring(0, hhFLB);

            //var gameTypes = ["Hold'em No Limit", ]
            //Test without array first, add array with for loop later (V1.2), return i + 1

            //Use last index of cos of home game name issue
            var isNLHE = hhFirstLine.lastIndexOf("Hold'em No Limit")
            if (isNLHE > 0) {
                return 1;
            }
            else {
                return 0;
            }
        }

        function hhTypeA() {
            //Simplest type, represents NLHE, PLO, etc

            //Find blind level
            var hhText = document.getElementById('hhInput').value;
            var hhFLB = hhText.indexOf("\n");
            var hhFirstLine = hhText.substring(0, hhFLB);

            //Use last index of cos of homegame name issue
            var blindSP = hhFirstLine.lastIndexOf("(");
            var blindEP = hhFirstLine.lastIndexOf(")");
            var hhBlinds = hhFirstLine.substring(blindSP + 1, blindEP);

            //Find button position
            var hhSLB = hhText.indexOf("\n", hhFLB + 1);
            var hhSecondLine = hhText.substring(hhFLB + 1, hhSLB);

            var buttonSP = hhSecondLine.indexOf("Seat #") + 6;
            var buttonEP = hhSecondLine.indexOf(" ", buttonSP);
            var hhButtonSeatNum = hhSecondLine.substring(buttonSP, buttonEP)

            //Players seated in the hand added to array
            var hhLineStart = hhSLB;
            var hhNewLine = "";

            var hhLineEnd = 0;
            var iCounter = 0;

            var hhPlayersInHand = [];
            var hhSeatsInHand = [];

            while(iCounter < 10){
                hhLineEnd = hhText.indexOf("\n", hhLineStart + 1);
                hhNewLine = hhText.substring(hhLineStart + 1, hhLineEnd);

                if (hhNewLine.indexOf("Seat ") == -1) {
                    iCounter += 100;

                }
                else {
                    //Check if player is at table but unable to play due to position
                    if (hhNewLine.indexOf(" out of hand ") == -1) {
                        hhPlayersInHand[iCounter] = hhNewLine.substring(hhNewLine.indexOf(":") + 2, hhNewLine.lastIndexOf("(") - 1);
                        hhSeatsInHand[iCounter] = hhNewLine.substring(hhNewLine.indexOf("Seat ") + 5, hhNewLine.indexOf(":"));
                        
                    }
                    else {
                        iCounter--;

                    }
                }
                hhLineStart = hhLineEnd;
                iCounter++;

            }

            //Assigns table positions to players
            var hhTablePositions = [];
            
            if (hhSeatsInHand.length == 2) {
                hhTablePositions = ["SB", "BB"]
            }
            else if (hhSeatsInHand.length == 3) {
                hhTablePositions = ["BTN", "SB", "BB"]
            }
            else if (hhSeatsInHand.length == 4) {
                hhTablePositions = ["BTN", "SB", "BB", "CO"]
            }
            else if (hhSeatsInHand.length == 5) {
                hhTablePositions = ["BTN", "SB", "BB", "UTG", "CO"]
            }
            else if (hhSeatsInHand.length == 6) {
                hhTablePositions = ["BTN", "SB", "BB", "UTG", "HJ", "CO"]
            }
            else if (hhSeatsInHand.length == 7) {
                hhTablePositions = ["BTN", "SB", "BB", "UTG", "LJ", "HJ", "CO"]
            }
            else if (hhSeatsInHand.length == 8) {
                hhTablePositions = ["BTN", "SB", "BB", "UTG", "UTG+1", "LJ", "HJ", "CO"]
            }
            else if (hhSeatsInHand.length == 9) {
                hhTablePositions = ["BTN", "SB", "BB", "UTG", "UTG+1", "UTG+2", "LJ", "HJ", "CO"]
            }
            else if (hhSeatsInHand.length == 10) {
                hhTablePositions = ["BTN", "SB", "BB", "UTG", "UTG+1", "UTG+2", "UTG+3", "LJ", "HJ", "CO"]
            }
            else {
                document.getElementById('hhOutput').value = "An error has occured. Please try again.";
                return;
            }

            var hhActualTP = [];
            var iCounter = 0;
            jCounter = hhButtonSeatNum - 1;

            while (iCounter != hhTablePositions.length) {
                hhActualTP[jCounter] = hhTablePositions[iCounter];

                jCounter++;
                if (jCounter == hhTablePositions.length) {
                    jCounter = 0;
                }

                iCounter++;
            }

            //Replace all player names with player positions
            for (i = 0; i < hhActualTP.length; i++) {
                hhText = hhText.replace(new RegExp(hhPlayersInHand[i], "g"), hhActualTP[i])
            }

            //Delete all chat messages
            hhLineStart = 0;
            hhLineEnd = hhText.indexOf("\n") + 1;

            while (hhLineEnd != 0) {
                hhNewLine = hhText.substring(hhLineStart, hhLineEnd);

                if (hhNewLine.indexOf(" said, ") == -1) {
                    hhLineStart = hhLineEnd;
                    hhLineEnd = hhText.indexOf("\n", hhLineEnd) + 1;

                }
                else {
                    hhText = hhText.replace(hhNewLine, "");
                    hhLineEnd = hhText.indexOf("\n", hhLineStart) + 1;

                }

            }

            //Calculate Ante
            var hhAnte = 0;
            var hhAnteFound = 0;
            hhLineStart = 0;
            hhLineEnd = hhText.indexOf("\n") + 1;

            while (hhLineEnd != 0) {
                hhNewLine = hhText.substring(hhLineStart, hhLineEnd);

                if (hhNewLine.indexOf("ante ") == -1) {
                    hhLineStart = hhLineEnd;
                    hhLineEnd = hhText.indexOf("\n", hhLineEnd) + 1;

                    if (hhAnteFound == 1) {
                        hhNewLine = 0;
                    }
                }
                else {
                    hhNewLine = hhNewLine.concat(" ");
                    if (hhNewLine.substring(hhNewLine.indexOf("ante ") + 5, hhNewLine.indexOf(" ", hhNewLine.indexOf("ante ") + 5)) > hhAnte) {
                        hhAnte = hhNewLine.substring(hhNewLine.indexOf("ante ") + 5, hhNewLine.indexOf(" ", hhNewLine.indexOf("ante ") + 5)).replace("\n", "");
                    }

                    hhLineStart = hhLineEnd;
                    hhLineEnd = hhText.indexOf("\n", hhLineEnd) + 1;

                }

            }



            
            
            //Temp Output 1
            document.getElementById('hhOutput').value = document.getElementById('hhOutput').value.concat(hhText, "\n\nBlinds: ", hhBlinds, ", Ante: ", hhAnte, ", Button: ", hhButtonSeatNum, "\n");
        }
    
    </script>

    <p>Enter Hand History Here: &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp; Recieve Converted Version Here:</p>
    <textarea name="hhInput" id="hhInput" rows="30" cols="60"></textarea>
    <input type="submit" value="Submit" onclick="hhConvert()"/>
    <textarea name="hhOutput" id="hhOutput" rows="30" cols="60"></textarea>

</body>
</html>
