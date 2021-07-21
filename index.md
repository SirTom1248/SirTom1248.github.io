

<!DOCTYPE html>
<html>
<!--Copyright, Christopher Wathen, 2020, All Rights Reserved-->

<head>
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link href='https://fonts.googleapis.com/css?family=Roboto Condensed' rel='stylesheet'>

    <style>
        body {
            font-size: 18px;
            font-family: 'Roboto Condensed';
            background-color: #2e2f30;
            color: white;
        }

        .title {
            font-size: 22px;
        }

        .disclaimer {
            font-size: 14px;
            margin: 1px 0px 0px 0px;
        }

        .button {
            background-color: #181a1b;
            border: 1px solid #c3c3c3;
            box-shadow: 0px 2px 2px #888888;
            color: white;
            padding: 10px 16px;
            text-align: center;
            text-decoration: none;
            display: inline-block;
            margin: 0px 20px 20px 0px;
            cursor: pointer;
            font-size: 22px;
            font-family: 'Roboto Condensed';
            border-radius: 5px;
            width: 138px;
        }

        #submit {
            margin: 20px 0px;
            width: 300px;
        }

        #copy-wrapper {
            position: relative;
        }

        #copied-notification {
            position: absolute;
            top: -10px;
            width: 122px;
            text-align: center;
            background-color: rgba(0,0,0,0.7);
            color: white;
            padding: .5rem;
            border-radius: 5px;
            display: none;
        }

            #copied-notification.visible {
                display: block;
                opacity: 1;
                transition: all 1s;
            }

                #copied-notification.visible.animating {
                    opacity: 0;
                    top: -20px;
                }

        .textarea {
            font-family: 'Roboto Condensed';
            font-size: 14px;
            border-radius: 10px;
            background-color: black;
            color: white;
            width: 95%;
            height: 300px;
            padding: .5em;
            box-shadow: 0px 2px 2px #888888;
        }

        .switch {
            position: relative;
            display: inline-block;
            width: 60px;
            height: 28px;
            margin: 0px 10px 0px 10px;
        }

            .switch input {
                opacity: 0;
                width: 0;
                height: 0;
            }

        .slider {
            position: absolute;
            cursor: pointer;
            top: 0;
            left: 0;
            right: 0;
            bottom: 0;
            background-color: black;
            -webkit-transition: .4s;
            transition: .4s;
        }

            .slider:before {
                position: absolute;
                content: "";
                height: 20px;
                width: 20px;
                left: 4px;
                bottom: 4px;
                background-color: #c3c3c3;
                -webkit-transition: .4s;
                transition: .4s;
            }

        input:focus + .slider {
            box-shadow: 0 0 1px #2196F3;
        }

        input:checked + .slider:before {
            -webkit-transform: translateX(32px);
            -ms-transform: translateX(32px);
            transform: translateX(32px);
        }

        /* Rounded sliders */
        .slider.round {
            border-radius: 23px;
        }

            .slider.round:before {
                border-radius: 50%;
            }
    </style>
</head>

<body>
    <script>
        //Global variables
        var hhDivisor = 1;

        //This is where standard functions are listed that are either called by the user or dictate what the program does when called by the user


        function hhReset() {
            //Resets input area

            document.getElementById('hhInput').value = "";
            return;
        }


        function hhCopy() {
            //Short copy function that allows the user to copy the hh when button is pressed

            var hhCopyText = document.querySelector("#hhInput");
            hhCopyText.select();
            document.execCommand("copy");
        }


        function hhErrorCheck() {
            //Basic error check to see if length of hh exceeds standard discord length

            if (document.getElementById('hhInput').value.length > 2000) {
                alert("Warning! After converting, the hand history you have entered exceeds 2000 characters! You may have trouble posting this hand in Discord!")
            }
        }


        function hhToggle() {
            //Short toggle function that changes value of global hhDivisor variable dependant on where user wants hh in chips or big blinds

            if (hhDivisor == 0) {
                hhDivisor = 1;
            }
            else {
                hhDivisor = 0;
            }
        }


        function hhConvert() {
            //This combines all the sub functions to convert the actual HH

            var hhText = document.getElementById('hhInput').value;
            document.getElementById('hhInput').value = "";

            //Short error check in case of leading whitespace
            while (hhText[0] == "\n") {
                if (hhText == "\n") {
                    document.getElementById('hhInput').value = "An error has occurred. Please try again.";
                    return;
                }
                else {
                    hhText = hhText.substring(1, hhText.length);
                }
            }

            //This declares which group each type num falls into
            var hhTypeNumCommon = [0, 1, 3, 4, 6, 7, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18];
            var hhTypeNumFixedLimit = [2, 5, 8, 19];

            var hhTypeNum = hhType(hhText);

            if (hhTypeNumCommon.includes(hhTypeNum) == true) {
                hhTypeCommon(hhText, hhTypeNumCommon.indexOf(hhTypeNum), hhDivisor);
            }
            else if (hhTypeNumFixedLimit.includes(hhTypeNum) == true) {
                hhTypeFixedLimit(hhText, hhTypeNumFixedLimit.indexOf(hhTypeNum), hhDivisor);
            }
            else {
                document.getElementById('hhInput').value = "An error has occurred. Please try again.";
                return;
            }

            if (document.getElementById('hhInput').value.indexOf("An error has occurred") == -1) {
                while (document.getElementById('hhInput').value.indexOf("Hero(") != -1) {
                    document.getElementById('hhInput').value = document.getElementById('hhInput').value.replace("Hero(", "Hero (")
                }

                document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat("\n\nConvert your hand history at <http://sirtom1248.github.io/>");
            }

            hhErrorCheck();
        }


        function hhType(hhText) {
            //This should detect whether NLHE/PLO/Stud/etc

            var hhFirstLine = hhText.substring(0, hhText.indexOf("\n"));
            var hhGameTypes = ["Hold'em No Limit", "Hold'em Pot Limit", "Hold'em Limit", "Omaha No Limit", "Omaha Pot Limit", "Omaha Limit", "Omaha Hi/Lo No Limit", "Omaha Hi/Lo Pot Limit", "Omaha Hi/Lo Limit", "5 Card Omaha No Limit", "5 Card Omaha Pot Limit", "5 Card Omaha Hi/Lo Pot Limit", "6 Card Omaha Pot Limit", "Swap Hold'em No Limit", "Swap Hold'em Pot Limit", "Fusion No Limit", "Fusion Pot Limit", "Showtime Hold'em No Limit", "Showtime Hold'em Pot Limit", "Showtime Hold'em Limit"];

            var i;
            for (i = hhGameTypes.length - 1; i >= 0; --i) {
                //Use last index of cos of home game name issue
                if (hhFirstLine.lastIndexOf(hhGameTypes[i]) != -1) {
                    return i;
                }
            }

            return -1;
        }


        //This is where sub functions are declared to be used for the various types


        function hhCardConverter(hhCards) {
            //This converts card names to discord recognisable ID's
            var hhCardValues = ["c", "d", "h", "s", "2", "3", "4", "5", "6", "7", "8", "9", "T", "J", "Q", "K", "A"];
            var hhCardIDs = [":shamrock:", ":large_blue_diamond:", ":hearts:", ":spoon:", ":two:", ":three:", ":four:", ":five:", ":six:", ":seven:", ":eight:", ":nine:", ":regional_indicator_t:", ":regional_indicator_j:", ":regional_indicator_q:", ":regional_indicator_k:", ":regional_indicator_a:"];
            var hhCardsOutput = "";

            var i;
            var j;
            for (i = 0; i < hhCards.length; ++i) {
                if (hhCards.charAt(i) == " ") {
                    hhCardsOutput = hhCardsOutput.concat(" ");
                }
                else {
                    for (j = 0; j < 17; ++j) {
                        if (hhCards.charAt(i) == hhCardValues[j]) {
                            hhCardsOutput = hhCardsOutput.concat(hhCardIDs[j]);
                            j = 17;
                        }
                    }
                }
            }

            return hhCardsOutput;
        }


        function hhGetBuyIn(hhText) {
            //Find Buy In
            var hhFirstLine = hhText.substring(0, hhText.indexOf("\n"));

            if (hhFirstLine.indexOf(": Tournament #") != -1 || hhFirstLine.indexOf("} Tournament #") != -1) {
                if (hhFirstLine.lastIndexOf("+") != -1) {
                    return hhFirstLine.substring(hhFirstLine.lastIndexOf(" ", hhFirstLine.lastIndexOf("+")) + 1, hhFirstLine.indexOf(" ", hhFirstLine.lastIndexOf("+")));
                }
                else if (hhFirstLine.indexOf(", Freeroll ") != -1) {
                    return "Freeroll";
                }
                else {
                    return "Buy In Not Found";
                }
            }
            else {
                return "";
            }
        }


        function hhGetBlinds(hhText) {
            //Find blind level
            var hhFirstLine = hhText.substring(0, hhText.indexOf("\n"));

            //Use last index of cos of homegame name issue
            return hhFirstLine.substring(hhFirstLine.lastIndexOf("(") + 1, hhFirstLine.lastIndexOf(")"));
        }


        function hhGetButtonPos(hhText) {
            //Find button position
            var hhSecondLine = hhText.substring(hhText.indexOf("\n") + 1, hhText.indexOf("\n", hhText.indexOf("\n") + 1));
            return hhSecondLine.substring(hhSecondLine.indexOf("Seat #") + 6, hhSecondLine.indexOf(" ", hhSecondLine.indexOf("Seat #") + 6));
        }


        function hhGetHeroName(hhText) {
            //Finds Name of Hero
            var hhLineStart = 0;
            var hhLineEnd = hhText.indexOf("\n") + 1;
            var hhNewLine;

            while (hhLineEnd != 0) {
                hhNewLine = hhText.substring(hhLineStart, hhLineEnd);

                if (hhNewLine.indexOf("Dealt to ") == -1) {
                    hhLineStart = hhLineEnd;
                    hhLineEnd = hhText.indexOf("\n", hhLineEnd) + 1;

                }
                else {
                    return hhNewLine.substring(hhNewLine.indexOf("Dealt to ") + 9, hhNewLine.lastIndexOf("[") - 1);
                }
            }

            return "No Hero Found";
        }


        function hhGetPlayersSeated(hhText, hhPlayersInHand, hhSeatsInHand) {
            //Players seated in the hand added to array
            var hhLineStart = hhText.indexOf("\n", hhText.indexOf("\n") + 1);
            var hhNewLine = "";

            var hhLineEnd = 0;
            var iCounter = 0;

            while (iCounter < 10) {
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

            return;
        }


        function hhGetTablePositions(hhText, hhSeatsInHand, hhActualTP, hhButtonSeatNum) {
            //Assigns table positions to players
            var hhTablePositions;

            if (hhSeatsInHand.length == 2) {
                hhTablePositions = ["SB", "BB"];
            }
            else if (hhSeatsInHand.length == 3) {
                hhTablePositions = ["BTN", "SB", "BB"];
            }
            else if (hhSeatsInHand.length == 4) {
                hhTablePositions = ["BTN", "SB", "BB", "CO"];
            }
            else if (hhSeatsInHand.length == 5) {
                hhTablePositions = ["BTN", "SB", "BB", "UTG", "CO"];
            }
            else if (hhSeatsInHand.length == 6) {
                hhTablePositions = ["BTN", "SB", "BB", "UTG", "HJ", "CO"];
            }
            else if (hhSeatsInHand.length == 7) {
                hhTablePositions = ["BTN", "SB", "BB", "UTG", "LJ", "HJ", "CO"];
            }
            else if (hhSeatsInHand.length == 8) {
                hhTablePositions = ["BTN", "SB", "BB", "UTG", "UTG+1", "LJ", "HJ", "CO"];
            }
            else if (hhSeatsInHand.length == 9) {
                hhTablePositions = ["BTN", "SB", "BB", "UTG", "UTG+1", "UTG+2", "LJ", "HJ", "CO"];
            }
            else if (hhSeatsInHand.length == 10) {
                hhTablePositions = ["BTN", "SB", "BB", "UTG", "UTG+1", "UTG+2", "UTG+3", "LJ", "HJ", "CO"];
            }
            else {
                document.getElementById('hhInput').value = "An error has occured. Please try again.";
                return;
            }

            var i;
            var hhRelativeBtnNum = 0;

            for (i = 0; i < hhTablePositions.length; ++i) {
                if (hhButtonSeatNum == hhSeatsInHand[i]) {
                    hhRelativeBtnNum = i;
                    i += 10;
                }
            }

            var iCounter = 0;
            var jCounter = hhRelativeBtnNum;

            while (iCounter != hhTablePositions.length) {
                hhActualTP[jCounter] = hhTablePositions[iCounter];

                jCounter++;
                if (jCounter == hhTablePositions.length) {
                    jCounter = 0;
                }

                iCounter++;
            }

            return;
        }


        function hhReplacePlayerNames(hhText, hhActualTP, hhPlayersInHand, hhHero, hhHeroPosTS) {
            //Replace all player names with player positions
            var i, TempStore;

            for (i = 0; i < hhActualTP.length; ++i) {
                while (hhText.indexOf(hhPlayersInHand[i]) != -1) {
                    if (hhPlayersInHand[i] == hhHero) {
                        TempStore = "Hero(";
                        hhText = hhText.replace(hhPlayersInHand[i], TempStore.concat(hhActualTP[i], ")"));
                        hhHeroPosTS[0] = hhActualTP[i];
                    }
                    else {
                        hhText = hhText.replace(hhPlayersInHand[i], hhActualTP[i]);
                    }
                }
            }

            return hhText;
        }


        function hhDeleteChatMsg(hhText) {
            //Delete all chat messages
            var hhLineStart = 0;
            var hhLineEnd = hhText.indexOf("\n") + 1;
            var hhNewLine;

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

            return hhText;
        }


        function hhDeleteCurrencyLabels(hhText) {
            //Delete all occurances of currency labels
            var hhNewLine, hhTempNewLine;
            var hhLineStart = 0;
            var hhLineEnd = hhText.indexOf("\n") + 1;

            while (hhLineEnd != 0) {
                hhNewLine = hhText.substring(hhLineStart, hhLineEnd);

                if (hhNewLine.indexOf("bounty") == -1) {
                    hhTempNewLine = hhNewLine.replace(/\$/g, "");
                    hhTempNewLine = hhTempNewLine.replace(/\£/g, "");
                    hhTempNewLine = hhTempNewLine.replace(/\€/g, "");
                    hhText = hhText.replace(hhNewLine, hhTempNewLine);
                }

                hhLineStart = hhLineEnd;
                hhLineEnd = hhText.indexOf("\n", hhLineEnd) + 1;
            }

            return hhText;
        }


        function hhGetAnteAndFAIAnte(hhText, hhAnteFAIStack, hhAnteFAIPlayer, hhActualTP) {
            //Calculate Ante and Players Forced All In Due To Ante
            var hhAnte = 0;
            var hhAnteCounter = 0;
            var hhAnteCounterFAI = 0;
            var hhAnteFound = 0;
            var hhLineStart = 0;
            var hhLineEnd = hhText.indexOf("\n") + 1;
            var hhNewLine;

            while (hhLineEnd != 0) {
                hhNewLine = hhText.substring(hhLineStart, hhLineEnd);

                if (hhNewLine.indexOf("ante ") == -1) {
                    hhLineStart = hhLineEnd;
                    hhLineEnd = hhText.indexOf("\n", hhLineEnd) + 1;

                    if (hhAnteFound == 1) {
                        hhLineEnd = 0;
                    }
                }
                else {
                    if (hhNewLine.indexOf("all-in") == -1) {
                        hhNewLine = hhNewLine.concat(" ");
                        hhAnte = Number(hhNewLine.substring(hhNewLine.indexOf("ante ") + 5, hhNewLine.indexOf(" ", hhNewLine.indexOf("ante ") + 5)).replace("\n", ""), 10);

                        if (isNaN(hhAnte) == true) {
                            hhAnte = Number(hhNewLine.substring(hhNewLine.indexOf("ante ") + 6, hhNewLine.indexOf(" ", hhNewLine.indexOf("ante ") + 5)).replace("\n", ""), 10);
                        }

                    }
                    else {
                        hhAnteFAIStack[hhAnteCounterFAI] = Number(hhNewLine.substring(hhNewLine.indexOf("ante ") + 5, hhNewLine.indexOf(" ", hhNewLine.indexOf("ante ") + 5)).replace("\n", ""), 10);
                        hhAnteFAIPlayer[hhAnteCounterFAI] = hhActualTP[hhAnteCounter];

                        hhAnteCounterFAI++;
                    }

                    hhAnteCounter++;
                    hhAnteFound = 1;
                    hhLineStart = hhLineEnd;
                    hhLineEnd = hhText.indexOf("\n", hhLineEnd) + 1;
                }

            }

            return [hhAnte, hhAnteCounter, hhAnteCounterFAI];
        }


        function hhGetStacksAndBounties(hhText, hhStacks, hhBounties, hhSatOut, hhActualTP) {
            //Store stacks and bounties
            var hhStacksCounter = 0;
            var hhStacksFound = 0;
            var hhLineStart = 0;
            var hhLineEnd = hhText.indexOf("\n") + 1;
            var hhNewLine;

            while (hhLineEnd != 0) {
                hhNewLine = hhText.substring(hhLineStart, hhLineEnd);

                if (hhNewLine.indexOf(" out of hand ") == -1) {
                    if (hhNewLine.indexOf(" in chips") == -1) {
                        hhLineStart = hhLineEnd;
                        hhLineEnd = hhText.indexOf("\n", hhLineEnd) + 1;

                        if (hhStacksFound == 1) {
                            hhLineEnd = 0;
                        }
                    }

                    else {
                        if (isNaN(hhNewLine.substring(hhNewLine.lastIndexOf("(") + 1, hhNewLine.indexOf(" ", hhNewLine.lastIndexOf("(")))) == false) {
                            hhStacks[hhStacksCounter] = hhNewLine.substring(hhNewLine.lastIndexOf("(") + 1, hhNewLine.indexOf(" ", hhNewLine.lastIndexOf("(")));
                        }
                        else {
                            hhStacks[hhStacksCounter] = hhNewLine.substring(hhNewLine.lastIndexOf("(") + 2, hhNewLine.indexOf(" ", hhNewLine.lastIndexOf("(")));
                        }

                        if (hhNewLine.indexOf(" bounty") != -1) {
                            hhBounties[hhStacksCounter] = hhNewLine.substring(hhNewLine.lastIndexOf(", ") + 2, hhNewLine.lastIndexOf(" bounty"));
                        }

                        if (hhNewLine.indexOf("is sitting out") != -1) {
                            var hhTempNewLine;
                            var hhTempLineEnd = hhLineEnd;

                            while (hhTempLineEnd != 0) {
                                hhTempNewLine = hhText.substring(hhTempLineEnd, hhText.indexOf("\n", hhTempLineEnd) + 1);

                                if (hhTempNewLine.indexOf(hhActualTP[hhStacksCounter]) != -1) {
                                    if (hhTempNewLine.indexOf("returned") != -1) {
                                        hhSatOut[hhStacksCounter] = 0;
                                        hhTempLineEnd = 0;
                                    }
                                    else if (hhTempNewLine.indexOf("folds") != -1) {
                                        hhSatOut[hhStacksCounter] = 1;
                                        hhTempLineEnd = 0;
                                    }
                                    else {
                                        hhTempLineEnd = hhText.indexOf("\n", hhTempLineEnd) + 1;
                                    }
                                }
                                else {
                                    hhTempLineEnd = hhText.indexOf("\n", hhTempLineEnd) + 1;
                                }
                            }
                        }
                        else {
                            hhSatOut[hhStacksCounter] = 0;
                        }

                        hhStacksCounter++;
                        hhStacksFound = 1;
                        hhLineStart = hhLineEnd;
                        hhLineEnd = hhText.indexOf("\n", hhLineEnd) + 1;

                    }
                }
                else {
                    hhLineStart = hhLineEnd;
                    hhLineEnd = hhText.indexOf("\n", hhLineEnd) + 1;
                }

            }

            return;
        }


        function hhGetTheorAndActBlinds(hhText, hhBlinds) {
            //Calculate actual and theoretical blinds
            hhBlinds = hhBlinds.concat(" ");

            var hhTheorSB = Number(hhBlinds.substring(0, hhBlinds.indexOf("/")));
            var hhTheorBB = Number(hhBlinds.substring(hhBlinds.indexOf("/") + 1, hhBlinds.indexOf(" ")));
            var hhCurrency = "";

            if (isNaN(hhTheorSB) == true) {
                hhCurrency = hhBlinds.substring(0, 1);
                hhTheorSB = Number(hhBlinds.substring(1, hhBlinds.indexOf("/")));
            }

            if (isNaN(hhTheorBB) == true) {
                hhTheorBB = Number(hhBlinds.substring(hhBlinds.indexOf("/") + 2, hhBlinds.indexOf(" ")));
            }

            var hhActualSB = 0;
            var hhActualBB = 0;

            var hhSBFound = 0;
            var hhBBFound = 0;

            var hhBBCounter = 0;
            var hhBringInString = "";

            var hhLineStart = 0;
            var hhLineEnd = hhText.indexOf("\n") + 1;
            var hhNewLine;

            while (hhLineEnd != 0) {
                hhNewLine = hhText.substring(hhLineStart, hhLineEnd);

                if (hhNewLine.indexOf("out of hand") == -1) {
                    if (hhNewLine.indexOf("blind") == -1) {
                        hhLineStart = hhLineEnd;
                        hhLineEnd = hhText.indexOf("\n", hhLineEnd) + 1;

                        if (hhSBFound == 1 || hhBBFound == 1) {
                            hhLineEnd = 0;
                        }
                    }

                    else {
                        if (hhNewLine.indexOf("all-in") != -1) {
                            if (hhNewLine.indexOf("small") != -1) {
                                hhActualSB = hhNewLine.substring(hhNewLine.indexOf("blind ") + 6, hhNewLine.indexOf(" ", hhNewLine.indexOf("blind ") + 6));
                            }
                            else if (hhNewLine.indexOf("big") != -1) {
                                hhActualBB = hhNewLine.substring(hhNewLine.indexOf("blind ") + 6, hhNewLine.indexOf(" ", hhNewLine.indexOf("blind ") + 6));
                            }
                        }

                        if (hhNewLine.indexOf("small") != -1) {
                            hhSBFound = 1;
                        }
                        else if (hhNewLine.indexOf("big") != -1) {
                            hhBBFound = 1;
                            hhBBCounter++;

                            if (hhNewLine.substring(0, hhNewLine.indexOf(" ")).replace(":", "") != "BB") {
                                hhBringInString = hhBringInString.concat(hhNewLine.substring(0, hhNewLine.indexOf(" ")).replace(":", ""), ", ");
                            }

                        }

                        hhLineStart = hhLineEnd;
                        hhLineEnd = hhText.indexOf("\n", hhLineEnd) + 1;
                    }
                }
                else {
                    hhLineStart = hhLineEnd;
                    hhLineEnd = hhText.indexOf("\n", hhLineEnd) + 1;
                }
            }

            return [hhActualBB, hhActualSB, hhTheorBB, hhTheorSB, hhBBFound, hhSBFound, hhBBCounter, hhCurrency, hhBringInString];
        }


        function hhGetRake(hhText, hhDivisor, hhCurrency, hhInBBText) {
            if (hhText.substring(hhText.indexOf("\n", hhText.indexOf(" | Rake ")) - 1, hhText.indexOf("\n", hhText.indexOf(" | Rake "))) != " ") {
                if (isNaN(hhText.substring(hhText.indexOf(" | Rake ") + 8, hhText.indexOf("\n", hhText.indexOf(" | Rake ")) - 1)) == false) {
                    return hhChipsToBBConverter(hhText.substring(hhText.indexOf(" | Rake ") + 8, hhText.indexOf("\n", hhText.indexOf(" | Rake "))), hhDivisor, hhCurrency, hhInBBText);
                }
                else {
                    return hhChipsToBBConverter(hhText.substring(hhText.indexOf(" | Rake ") + 9, hhText.indexOf("\n", hhText.indexOf(" | Rake "))), hhDivisor, hhCurrency, hhInBBText);
                }
            }
            else {
                if (isNaN(hhText.substring(hhText.indexOf(" | Rake ") + 8, hhText.indexOf("\n", hhText.indexOf(" | Rake ")) - 1)) == false) {
                    return hhChipsToBBConverter(hhText.substring(hhText.indexOf(" | Rake ") + 8, hhText.indexOf("\n", hhText.indexOf(" | Rake ")) - 1), hhDivisor, hhCurrency, hhInBBText);
                }
                else {
                    return hhChipsToBBConverter(hhText.substring(hhText.indexOf(" | Rake ") + 9, hhText.indexOf("\n", hhText.indexOf(" | Rake ")) - 1), hhDivisor, hhCurrency, hhInBBText);
                }
            }
        }


        function hhChipsToBBConverter(x, hhDivisor, hhCurrency, hhInBBText) {
            if (hhCurrency == "" && hhInBBText == "") {
                return Math.round(((Number(x) / hhDivisor) + Number.EPSILON) * 100) / 100;
            }
            else {
                if (Number.isInteger(Math.round(((Number(x) / hhDivisor) + Number.EPSILON) * 100) / 100) == true) {
                    return Math.round(((Number(x) / hhDivisor) + Number.EPSILON) * 100) / 100;
                }
                else {
                    return (Math.round(((Number(x) / hhDivisor) + Number.EPSILON) * 100) / 100).toFixed(2);
                }
            }
        }


        function hhNormalizeOutputStacks(hhActualTP, hhStacks, hhBounties, hhCurrency, hhInBBText, hhSatOut, hhHeroPos) {
            //Normalize "starting" position
            var hhPosNormalizer = 0;
            var hhPosNormFound = 0;

            while (hhPosNormFound == 0) {
                if (hhActualTP.length == 2) {
                    if (hhActualTP[hhPosNormalizer] == "SB") {
                        hhPosNormFound = 1;
                    }
                }
                else if (hhActualTP.length == 3) {
                    if (hhActualTP[hhPosNormalizer] == "BTN") {
                        hhPosNormFound = 1;
                    }
                }
                else if (hhActualTP.length == 4) {
                    if (hhActualTP[hhPosNormalizer] == "CO") {
                        hhPosNormFound = 1;
                    }
                }
                else {
                    if (hhActualTP[hhPosNormalizer] == "UTG") {
                        hhPosNormFound = 1;
                    }
                }

                hhPosNormalizer++;
            }

            hhPosNormalizer--;
            var hhNormalizedi;


            //Output Stack sizes
            var i;
            for (i = 0; i < hhStacks.length; ++i) {
                hhNormalizedi = hhPosNormalizer + i;

                if (hhNormalizedi == hhStacks.length) {
                    hhPosNormalizer = -i;
                    hhNormalizedi = hhPosNormalizer + i;
                }

                if (hhHeroPos != hhActualTP[hhNormalizedi]) {
                    if (hhBounties.length == 0) {
                        document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat("> ", hhActualTP[hhNormalizedi], " - ", hhCurrency, hhStacks[hhNormalizedi], hhInBBText, "\n");
                        if (hhSatOut[hhNormalizedi] == 1) {
                            document.getElementById('hhInput').value = document.getElementById('hhInput').value.substring(0, document.getElementById('hhInput').value.length - 1);
                            document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat(" - Player is sat out :zzz:\n")
                        }
                    }
                    else {
                        document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat("> ", hhActualTP[hhNormalizedi], " - ", hhCurrency, hhStacks[hhNormalizedi], hhInBBText, " - ", hhBounties[hhNormalizedi], " :dart:", "\n");
                        if (hhSatOut[hhNormalizedi] == 1) {
                            document.getElementById('hhInput').value = document.getElementById('hhInput').value.substring(0, document.getElementById('hhInput').value.length - 1);
                            document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat(" - Player is sat out :zzz:\n")
                        }
                    }
                }
                else {
                    if (hhBounties.length == 0) {
                        document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat("> Hero(", hhActualTP[hhNormalizedi], ") - ", hhCurrency, hhStacks[hhNormalizedi], hhInBBText, "\n");
                        if (hhSatOut[hhNormalizedi] == 1) {
                            document.getElementById('hhInput').value = document.getElementById('hhInput').value.substring(0, document.getElementById('hhInput').value.length - 1);
                            document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat(" - Player is sat out :zzz:\n")
                        }
                    }
                    else {
                        document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat("> Hero(", hhActualTP[hhNormalizedi], ") - ", hhCurrency, hhStacks[hhNormalizedi], hhInBBText, " - ", hhBounties[hhNormalizedi], " :dart:", "\n");
                        if (hhSatOut[hhNormalizedi] == 1) {
                            document.getElementById('hhInput').value = document.getElementById('hhInput').value.substring(0, document.getElementById('hhInput').value.length - 1);
                            document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat(" - Player is sat out :zzz:\n")
                        }
                    }
                }
            }

            document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat("\n> ");

            return;
        }


        function hhOutputIntro(hhTypeName, hhAnteCounter, hhCurrency, hhTheorSB, hhInBBText, hhTheorBB, hhInBBText, hhAnte, hhAnteFAIStack, hhBuyIn) {
            //Displays beginning of Output
            if (hhBuyIn != "") {
                document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat("> Buy In: ", hhBuyIn, "\n");
            }

            document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat("> Game Type: ", hhTypeName, "\n", "> Ante: ");

            if (hhAnteCounter == 0) {
                document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat("None | Blinds: ", hhCurrency, hhTheorSB, hhInBBText, " / ", hhCurrency, hhTheorBB, hhInBBText, "\n\n> Stacks:\n");
            }
            else if (hhAnte == 0) {
                document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat(hhCurrency, Math.max.apply(Math, hhAnteFAIStack), hhInBBText, " | Blinds: ", hhCurrency, hhTheorSB, hhInBBText, " / ", hhCurrency, hhTheorBB, hhInBBText, "\n\n> Stacks:\n");
            }
            else {
                document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat(hhCurrency, hhAnte, hhInBBText, " | Blinds: ", hhCurrency, hhTheorSB, hhInBBText, " / ", hhCurrency, hhTheorBB, hhInBBText, "\n\n> Stacks:\n");
            }

            return;
        }


        function hhOutputBlindsAntes(hhAnteCounter, hhAnteCounterFAI, hhAnteFAIPlayer, hhActualTP, hhCurrency, hhAnteFAIStack, hhInBBText, hhAnte, hhBBCounter, hhTheorBB, hhSBFound, hhActualSB, hhTheorSB, hhBBFound, hhActualBB, hhBringInString, hhHeroPos) {
            //Displays posted blinds and antes
            if (hhAnteCounter - hhAnteCounterFAI <= 1) {
                var hhAnteOutputCounter = 0;

                for (i = 0; i < hhAnteCounter; ++i) {
                    if (hhAnteFAIPlayer[hhAnteOutputCounter] == hhActualTP[i]) {
                        if (hhHeroPos != hhActualTP[i]) {
                            document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat(hhActualTP[i], " posts ante ", hhCurrency, hhAnteFAIStack[hhAnteOutputCounter], hhInBBText, " and is all-in, ");
                            hhAnteOutputCounter++;
                        }
                        else {
                            document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat("Hero(", hhActualTP[i], ") posts ante ", hhCurrency, hhAnteFAIStack[hhAnteOutputCounter], hhInBBText, " and is all-in, ");
                            hhAnteOutputCounter++;
                        }
                    }
                    else {
                        if (hhHeroPos != hhActualTP[i]) {
                            document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat(hhActualTP[i], " posts ante ", hhCurrency, hhAnte, hhInBBText, ", ");
                        }
                        else {
                            document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat("Hero(", hhActualTP[i], ") posts ante ", hhCurrency, hhAnte, hhInBBText, ", ");
                        }
                    }
                }
            }
            else {
                document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat((hhAnteCounter - hhAnteCounterFAI), " players post ante ", hhCurrency, hhAnte, hhInBBText, ", ");

                for (i = 0; i < hhAnteCounterFAI; ++i) {
                    if (hhHeroPos != hhActualTP[i]) {
                        document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat(hhAnteFAIPlayer[i], " posts ante ", hhCurrency, hhAnteFAIStack[i], hhInBBText, " and is all-in, ");
                    }
                    else {
                        document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat("Hero(", hhAnteFAIPlayer[i], ") posts ante ", hhCurrency, hhAnteFAIStack[i], hhInBBText, " and is all-in, ");
                    }
                }
            }

            if (hhBBCounter == 2) {
                document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat(hhBringInString.substring(0, hhBringInString.length - 2), " brings in ", hhCurrency, hhTheorBB, hhInBBText, " to join the hand, ");
            }
            else if (hhBBCounter > 2) {
                document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat(hhBringInString.substring(0, hhBringInString.length - 2), " bring in ", hhCurrency, hhTheorBB, hhInBBText, " to join the hand, ");
            }

            if (hhHeroPos != "SB") {
                if (hhSBFound == 1 && hhActualSB != 0) {
                    document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat("SB posts small blind ", hhCurrency, hhActualSB, hhInBBText, " and is all-in, ");
                }
                else if (hhSBFound == 1 && hhActualSB == 0) {
                    document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat("SB posts small blind ", hhCurrency, hhTheorSB, hhInBBText, ", ");
                }
            }
            else {
                if (hhSBFound == 1 && hhActualSB != 0) {
                    document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat("Hero(SB) posts small blind ", hhCurrency, hhActualSB, hhInBBText, " and is all-in, ");
                }
                else if (hhSBFound == 1 && hhActualSB == 0) {
                    document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat("Hero(SB) posts small blind ", hhCurrency, hhTheorSB, hhInBBText, ", ");
                }
            }

            if (hhHeroPos != "BB") {
                if (hhBBFound == 1 && hhActualBB != 0) {
                    document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat("BB posts big blind ", hhCurrency, hhActualBB, hhInBBText, " and is all-in\n\n> **PREFLOP**\n> ");
                }
                else if (hhBBFound == 1 && hhActualBB == 0) {
                    document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat("BB posts big blind ", hhCurrency, hhTheorBB, hhInBBText, "\n\n> **PREFLOP**\n> ");
                }
                else {
                    document.getElementById('hhInput').value = document.getElementById('hhInput').value.substring(0, document.getElementById('hhInput').value.length - 2);
                    document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat("\n\n> **PREFLOP**\n> ");
                }
            }
            else {
                if (hhBBFound == 1 && hhActualBB != 0) {
                    document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat("Hero(BB) posts big blind ", hhCurrency, hhActualBB, hhInBBText, " and is all-in\n\n> **PREFLOP**\n> ");
                }
                else if (hhBBFound == 1 && hhActualBB == 0) {
                    document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat("Hero(BB) posts big blind ", hhCurrency, hhTheorBB, hhInBBText, "\n\n> **PREFLOP**\n> ");
                }
                else {
                    document.getElementById('hhInput').value = document.getElementById('hhInput').value.substring(0, document.getElementById('hhInput').value.length - 2);
                    document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat("\n\n> **PREFLOP**\n> ");
                }
            }

            return;
        }


        function hhBBConverter(hhText, hhDivisor, hhCurrency, hhInBBText) {
            //Divide neccessary data by hhDivisor so BB conversion elligible
            var hhTempStore, hhTempStoreTwo, hhNewLine;
            var hhLineStart = 0;
            var hhLineEnd = hhText.indexOf("\n") + 1;

            while (hhLineEnd != 0) {
                hhNewLine = hhText.substring(hhLineStart, hhLineEnd);

                if (hhNewLine.indexOf("Uncalled bet") != -1) {
                    hhTempStore = String(hhChipsToBBConverter(hhNewLine.substring(hhNewLine.indexOf("(") + 1, hhNewLine.indexOf(")")), hhDivisor, hhCurrency, hhInBBText));
                    hhTempStore = hhTempStore.concat(hhInBBText);
                    hhTempNewLine = hhNewLine.replace(hhNewLine.substring(hhNewLine.indexOf("(") + 1, hhNewLine.indexOf(")")), hhTempStore);
                    hhText = hhText.replace(hhNewLine, hhTempNewLine);
                }
                else if (hhNewLine.indexOf("cashed out ") != -1) {
                    hhTempStore = String(hhChipsToBBConverter(hhNewLine.substring(hhNewLine.indexOf("for ") + 4, hhNewLine.length - 1), hhDivisor, hhCurrency, hhInBBText));
                    hhTempStore = hhTempStore.concat(hhInBBText);
                    hhTempNewLine = hhNewLine.replace(hhNewLine.substring(hhNewLine.indexOf("for ") + 4, hhNewLine.length - 1), hhTempStore);
                    hhText = hhText.replace(hhNewLine, hhTempNewLine);
                }
                else if (hhNewLine.indexOf("raises") != -1) {
                    hhTempStoreTwo = hhNewLine.substring(0, hhNewLine.length - 1);
                    hhTempStoreTwo = hhTempStoreTwo.concat(" \n");
                    hhTempStore = String(hhChipsToBBConverter(hhTempStoreTwo.substring(hhTempStoreTwo.indexOf("to") + 3, hhTempStoreTwo.indexOf(" ", hhTempStoreTwo.indexOf("to") + 3)), hhDivisor, hhCurrency, hhInBBText));
                    hhTempStore = hhTempStore.concat(hhInBBText);
                    hhTempNewLine = hhNewLine.replace(hhTempStoreTwo.substring(hhTempStoreTwo.indexOf("to") + 3, hhTempStoreTwo.indexOf(" ", hhTempStoreTwo.indexOf("to") + 3)), hhTempStore);
                    hhText = hhText.replace(hhNewLine, hhTempNewLine);
                }
                else if (hhNewLine.indexOf("bets") != -1) {
                    hhTempStoreTwo = hhNewLine.substring(0, hhNewLine.length - 1);
                    hhTempStoreTwo = hhTempStoreTwo.concat(" \n");
                    hhTempStore = String(hhChipsToBBConverter(hhTempStoreTwo.substring(hhTempStoreTwo.indexOf("bets") + 5, hhTempStoreTwo.indexOf(" ", hhTempStoreTwo.indexOf("bets") + 5)), hhDivisor, hhCurrency, hhInBBText));
                    hhTempStore = hhTempStore.concat(hhInBBText);
                    hhTempNewLine = hhNewLine.replace(hhTempStoreTwo.substring(hhTempStoreTwo.indexOf("bets") + 5, hhTempStoreTwo.indexOf(" ", hhTempStoreTwo.indexOf("bets") + 5)), hhTempStore);
                    hhText = hhText.replace(hhNewLine, hhTempNewLine);
                }
                else if (hhNewLine.indexOf("calls") != -1 && hhNewLine.indexOf("is all-in") != -1) {
                    hhTempStore = String(hhChipsToBBConverter(hhNewLine.substring(hhNewLine.indexOf("calls") + 6, hhNewLine.indexOf(" ", hhNewLine.indexOf("calls") + 6)), hhDivisor, hhCurrency, hhInBBText));
                    hhTempStore = hhTempStore.concat(hhInBBText);
                    hhTempNewLine = hhNewLine.replace(hhNewLine.substring(hhNewLine.indexOf("calls") + 6, hhNewLine.indexOf(" ", hhNewLine.indexOf("calls") + 6)), hhTempStore);
                    hhText = hhText.replace(hhNewLine, hhTempNewLine);
                }
                else if (hhNewLine.indexOf("collected") != -1) {
                    hhTempStore = String(hhChipsToBBConverter(hhNewLine.substring(hhNewLine.indexOf("collected") + 10, hhNewLine.indexOf(" ", hhNewLine.indexOf("collected") + 10)), hhDivisor, hhCurrency, hhInBBText));
                    hhTempStore = hhTempStore.concat(hhInBBText);
                    hhTempNewLine = hhNewLine.replace(hhNewLine.substring(hhNewLine.indexOf("collected") + 10, hhNewLine.indexOf(" ", hhNewLine.indexOf("collected") + 10)), hhTempStore);
                    hhText = hhText.replace(hhNewLine, hhTempNewLine);
                }

                hhLineStart = hhLineEnd;
                hhLineEnd = hhText.indexOf("\n", hhLineEnd) + 1;
            }

            return hhText;
        }


        function hhPotSum(total, num) {
            return total + num;
        }


        function hhPotCalculator(hhText, hhActualTP, hhStacks, hhAnteFAIPlayer, hhAnteFAIStack, hhAnte, hhPotTotal, hhPotPreflop, hhPotFlop, hhPotTurn, hhPotRiver, hhActualBB, hhActualSB, hhTheorBB, hhTheorSB, hhBBFound, hhSBFound, hhBBCounter, hhBringInString, hhFoldTracker) {
            //Calculate Pot at every relevant point
            var hhTempHeroStore = "Hero(";

            for (i = 0; i < hhActualTP.length; ++i) {
                //Add antes to Pot Total
                if(hhAnteFAIPlayer.length == 0){
                    hhPotTotal[i] = hhAnte;
                }
                else{
                    for (j = 0; j < hhAnteFAIPlayer.length; ++j) {
                        if (hhActualTP[i] == hhAnteFAIPlayer[j]) {
                            hhPotTotal[i] = hhAnteFAIStack[j];
                        }
                    }

                    if (hhPotTotal[i] == 0) {
                        hhPotTotal[i] = hhAnte;
                    }
                }
            }

            if (hhBBCounter >= 2) {
                //Add bring ins to Pot Preflop
                for (i = 0; i < hhActualTP.length; ++i) {
                    if (hhBringInString.indexOf(hhActualTP[i].concat(",")) != -1 || hhBringInString.indexOf(hhTempHeroStore.concat(hhActualTP[i], "),")) != -1) {
                        hhPotPreflop[i] = hhTheorBB;
                    }
                }
            }

            if (hhSBFound == 1 && hhActualSB != 0) {
                //Add all in SB to Pot Preflop
                for (i = 0; i < hhActualTP.length; ++i) {
                    if (hhActualTP[i] == "SB") {
                        hhPotPreflop[i] = hhActualSB;
                        i = 11;
                    }
                }
            }
            else if (hhSBFound == 1 && hhActualSB == 0) {
                //Add SB to Pot Preflop
                for (i = 0; i < hhActualTP.length; ++i) {
                    if (hhActualTP[i] == "SB") {
                        hhPotPreflop[i] = hhTheorSB;
                        i = 11;
                    }
                }
            }

            if (hhBBFound == 1 && hhActualBB != 0) {
                //Add all in BB to Pot Preflop
                for (i = 0; i < hhActualTP.length; ++i) {
                    if (hhActualTP[i] == "BB") {
                        hhPotPreflop[i] = hhActualBB;
                        i = 11;
                    }
                }
            }
            else if (hhBBFound == 1 && hhActualBB == 0) {
                //Add BB to Pot Preflop
                for (i = 0; i < hhActualTP.length; ++i) {
                    if (hhActualTP[i] == "BB") {
                        hhPotPreflop[i] = hhTheorBB;
                        i = 11;
                    }
                }
            }

            var hhTempStore, hhTempStoreTwo, hhTempPosStore, hhNewLine, i, j;
            var hhLineStart = 0;
            var hhLineEnd = hhText.indexOf("\n") + 1;
            var hhTempStoreLocation = "Preflop";

            while (hhLineEnd != 0) {
                hhNewLine = hhText.substring(hhLineStart, hhLineEnd);

                if (hhNewLine.indexOf("Uncalled bet") != -1) {
                    hhTempStore = hhNewLine.substring(hhNewLine.indexOf("(") + 1, hhNewLine.indexOf(")"));
                    hhTempPosStore = hhNewLine.substring(hhNewLine.indexOf("to") + 2, hhNewLine.length - 1).replace(" ", "");

                    for (i = 0; i < hhActualTP.length; ++i) {
                        if (hhActualTP[i] == hhTempPosStore || hhTempHeroStore.concat(hhActualTP[i], ")") == hhTempPosStore) {
                            if (hhTempStoreLocation == "Preflop") {
                                if (isNaN(Number(hhTempStore)) == false) {
                                    hhPotPreflop[i] -= Number(hhTempStore);
                                }
                                else {
                                    hhPotPreflop[i] -= Number(hhTempStore.substring(1, hhTempStore.length));
                                }
                            }
                            else if (hhTempStoreLocation == "Flop") {
                                if (isNaN(Number(hhTempStore)) == false) {
                                    hhPotFlop[i] -= Number(hhTempStore);
                                }
                                else {
                                    hhPotFlop[i] -= Number(hhTempStore.substring(1, hhTempStore.length));
                                }
                            }
                            else if (hhTempStoreLocation == "Turn") {
                                if (isNaN(Number(hhTempStore)) == false) {
                                    hhPotTurn[i] -= Number(hhTempStore);
                                }
                                else {
                                    hhPotTurn[i] -= Number(hhTempStore.substring(1, hhTempStore.length));
                                }
                            }
                            else if (hhTempStoreLocation == "River") {
                                if (isNaN(Number(hhTempStore)) == false) {
                                    hhPotRiver[i] -= Number(hhTempStore);
                                }
                                else {
                                    hhPotRiver[i] -= Number(hhTempStore.substring(1, hhTempStore.length));
                                }
                            }

                            i = 11;
                        }
                    }
                }
                else if (hhNewLine.indexOf("raises") != -1) {
                    hhTempStore = hhNewLine.substring(hhNewLine.indexOf("to") + 3, hhNewLine.length).replace("\n", " ");
                    hhTempStore = hhTempStore.substring(0, hhTempStore.indexOf(" "));
                    hhTempPosStore = hhNewLine.substring(0, hhNewLine.indexOf(":"));

                    for (i = 0; i < hhActualTP.length; ++i) {
                        if (hhActualTP[i] == hhTempPosStore || hhTempHeroStore.concat(hhActualTP[i], ")") == hhTempPosStore) {
                            if (hhTempStoreLocation == "Preflop") {
                                if (isNaN(Number(hhTempStore)) == false) {
                                    hhPotPreflop[i] = Number(hhTempStore);
                                }
                                else {
                                    hhPotPreflop[i] = Number(hhTempStore.substring(1, hhTempStore.length));
                                }
                            }
                            else if (hhTempStoreLocation == "Flop") {
                                if (isNaN(Number(hhTempStore)) == false) {
                                    hhPotFlop[i] = Number(hhTempStore);
                                }
                                else {
                                    hhPotFlop[i] = Number(hhTempStore.substring(1, hhTempStore.length));
                                }
                            }
                            else if (hhTempStoreLocation == "Turn") {
                                if (isNaN(Number(hhTempStore)) == false) {
                                    hhPotTurn[i] = Number(hhTempStore);
                                }
                                else {
                                    hhPotTurn[i] = Number(hhTempStore.substring(1, hhTempStore.length));
                                }
                            }
                            else if (hhTempStoreLocation == "River") {
                                if (isNaN(Number(hhTempStore)) == false) {
                                    hhPotRiver[i] = Number(hhTempStore);
                                }
                                else {
                                    hhPotRiver[i] = Number(hhTempStore.substring(1, hhTempStore.length));
                                }
                            }

                            i = 11;
                        }
                    }
                }
                else if (hhNewLine.indexOf("bets") != -1) {
                    hhTempStore = hhNewLine.substring(hhNewLine.indexOf("bets") + 5, hhNewLine.length).replace("\n", " ");
                    hhTempStore = hhTempStore.substring(0, hhTempStore.indexOf(" "));
                    hhTempPosStore = hhNewLine.substring(0, hhNewLine.indexOf(":"));

                    for (i = 0; i < hhActualTP.length; ++i) {
                        if (hhActualTP[i] == hhTempPosStore || hhTempHeroStore.concat(hhActualTP[i], ")") == hhTempPosStore) {
                            if (hhTempStoreLocation == "Flop") {
                                if (isNaN(Number(hhTempStore)) == false) {
                                    hhPotFlop[i] = Number(hhTempStore);
                                }
                                else {
                                    hhPotFlop[i] = Number(hhTempStore.substring(1, hhTempStore.length));
                                }
                            }
                            else if (hhTempStoreLocation == "Turn") {
                                if (isNaN(Number(hhTempStore)) == false) {
                                    hhPotTurn[i] = Number(hhTempStore);
                                }
                                else {
                                    hhPotTurn[i] = Number(hhTempStore.substring(1, hhTempStore.length));
                                }
                            }
                            else if (hhTempStoreLocation == "River") {
                                if (isNaN(Number(hhTempStore)) == false) {
                                    hhPotRiver[i] = Number(hhTempStore);
                                }
                                else {
                                    hhPotRiver[i] = Number(hhTempStore.substring(1, hhTempStore.length));
                                }
                            }

                            i = 11;
                        }
                    }
                }
                else if (hhNewLine.indexOf("calls") != -1) {
                    hhTempStore = hhNewLine.substring(hhNewLine.indexOf("calls") + 6, hhNewLine.length).replace("\n", " ");
                    hhTempStore = hhTempStore.substring(0, hhTempStore.indexOf(" "));
                    hhTempPosStore = hhNewLine.substring(0, hhNewLine.indexOf(":"));

                    for (i = 0; i < hhActualTP.length; ++i) {
                        if (hhActualTP[i] == hhTempPosStore || hhTempHeroStore.concat(hhActualTP[i], ")") == hhTempPosStore) {
                            if (hhTempStoreLocation == "Preflop") {
                                if (isNaN(Number(hhTempStore)) == false) {
                                    hhPotPreflop[i] += Number(hhTempStore);
                                }
                                else {
                                    hhPotPreflop[i] += Number(hhTempStore.substring(1, hhTempStore.length));
                                }
                            }
                            else if (hhTempStoreLocation == "Flop") {
                                if (isNaN(Number(hhTempStore)) == false) {
                                    hhPotFlop[i] += Number(hhTempStore);
                                }
                                else {
                                    hhPotFlop[i] += Number(hhTempStore.substring(1, hhTempStore.length));
                                }
                            }
                            else if (hhTempStoreLocation == "Turn") {
                                if (isNaN(Number(hhTempStore)) == false) {
                                    hhPotTurn[i] += Number(hhTempStore);
                                }
                                else {
                                    hhPotTurn[i] += Number(hhTempStore.substring(1, hhTempStore.length));
                                }
                            }
                            else if (hhTempStoreLocation == "River") {
                                if (isNaN(Number(hhTempStore)) == false) {
                                    hhPotRiver[i] += Number(hhTempStore);
                                }
                                else {
                                    hhPotRiver[i] += Number(hhTempStore.substring(1, hhTempStore.length));
                                }
                            }

                            i = 11;
                        }
                    }
                }
                else if (hhNewLine.indexOf("folds") != -1) {
                    hhTempPosStore = hhNewLine.substring(0, hhNewLine.indexOf(":"));

                    for (i = 0; i < hhActualTP.length; ++i) {
                        if (hhActualTP[i] == hhTempPosStore || hhTempHeroStore.concat(hhActualTP[i], ")") == hhTempPosStore) {
                            if (hhTempStoreLocation == "Preflop") {
                                hhFoldTracker[i] = 1;
                            }
                            else if (hhTempStoreLocation == "Flop") {
                                hhFoldTracker[i] = 2;
                            }
                            else if (hhTempStoreLocation == "Turn") {
                                hhFoldTracker[i] = 3;
                            }

                            i = 11;
                        }
                    }
                }
                else if (hhNewLine.indexOf(" FLOP ") != -1) {
                    hhTempStoreLocation = "Flop";
                }
                else if (hhNewLine.indexOf(" TURN ") != -1) {
                    hhTempStoreLocation = "Turn";
                }
                else if (hhNewLine.indexOf(" RIVER ") != -1) {
                    hhTempStoreLocation = "River";
                }
                else if (hhNewLine.indexOf(" SHOW DOWN ") != -1) {
                    hhTempStoreLocation = "Show Down";
                }

                hhLineStart = hhLineEnd;
                hhLineEnd = hhText.indexOf("\n", hhLineEnd) + 1;
            }

            return;
        }


        function hhPotSummaryOutput(hhPotTotal, hhPotPreflop, hhPotFlop, hhPotTurn, hhPotRiver, hhStacks, hhCurrency, hhDivisor, hhInBBText) {
            //Calculates neccessary pot sizes to be output
            var i;

            var hhPotFlopOutput = "";
            var hhPotTurnOutput = "";
            var hhPotRiverOutput = "";
            var hhPotShowDownOutput = "";

            var hhPotAllInCount = 0;
            var hhPotAllInStacks = [0, 0, 0, 0, 0, 0, 0, 0, 0, 0];
            var hhPotTempStore = [0, 0, 0, 0, 0, 0, 0, 0, 0, 0];
            var hhPotTempStoreTwo = [0, 0, 0, 0, 0, 0, 0, 0, 0, 0];
            var hhPotSizes = [0, 0, 0, 0, 0, 0, 0, 0, 0];
            var hhSmallestAllIn;

            for (i = 0; i < hhPotTotal.length; ++i) {
                hhPotTotal[i] += Number(hhPotPreflop[i]);
            }

            for (i = 0; i < hhPotTotal.length; ++i) {
                if (hhChipsToBBConverter(hhPotTotal[i], hhDivisor, hhCurrency, hhInBBText) == hhStacks[i]) {
                    hhPotAllInCount++;
                    hhPotAllInStacks[i] = hhPotTotal[i];
                }
            }

            if(hhPotAllInCount == 0){
                hhPotFlopOutput = hhPotFlopOutput.concat(hhCurrency, hhChipsToBBConverter(hhPotTotal.reduce(hhPotSum), hhDivisor, hhCurrency, hhInBBText), hhInBBText);
            }
            else if (hhPotAllInCount > 0) {
                for (i = 0; i < hhPotTempStore.length; ++i) {
                    hhPotTempStore[i] = hhPotTotal[i];
                }

                while (Math.max(...hhPotAllInStacks) > 0) {
                    hhSmallestAllIn = Math.max(...hhPotAllInStacks);

                    for (i = 0; i < hhPotAllInStacks.length; ++i) {
                        if (hhPotAllInStacks[i] > 0 && hhPotAllInStacks[i] < hhSmallestAllIn) {
                            hhSmallestAllIn = hhPotAllInStacks[i];
                        }
                    }

                    for (i = 0; i < hhPotTempStore.length; ++i) {
                        hhPotTempStoreTwo[i] = Math.min(hhPotTempStore[i], hhSmallestAllIn);
                    }


                    i = 0;
                    while (hhPotSizes[i] != 0) {
                        i++;
                    }

                    hhPotSizes[i] = hhPotTempStoreTwo.reduce(hhPotSum);


                    for (i = 0; i < hhPotAllInStacks.length; ++i) {
                        hhPotAllInStacks[i] = Math.max(0, hhPotAllInStacks[i] - hhSmallestAllIn);
                        hhPotTempStore[i] = Math.max(0, hhPotTempStore[i] - hhSmallestAllIn);
                    }
                }

                i = 0;
                while (hhPotSizes[i] != 0) {
                    ++i;
                }

                hhPotSizes[i] = hhPotTempStore.reduce(hhPotSum);

                hhPotFlopOutput = hhPotFlopOutput.concat("Main Pot: ", hhCurrency, hhChipsToBBConverter(hhPotSizes[0], hhDivisor, hhCurrency, hhInBBText), hhInBBText, ", ");

                i = 1;
                while (hhPotSizes[i] > 0.0001) {
                    hhPotFlopOutput = hhPotFlopOutput.concat("Side Pot ", i, ": ", hhCurrency, hhChipsToBBConverter(hhPotSizes[i], hhDivisor, hhCurrency, hhInBBText), hhInBBText, ", ");
                    ++i;
                }

                hhPotFlopOutput = hhPotFlopOutput.substring(0, hhPotFlopOutput.length - 2);

                if (hhPotFlopOutput.indexOf("Side Pot ") == -1) {
                    hhPotFlopOutput = hhPotFlopOutput.replace("Main Pot: ", "");
                }
            }


            hhPotAllInCount = 0;
            hhPotAllInStacks = [0, 0, 0, 0, 0, 0, 0, 0, 0, 0];
            hhPotTempStore = [0, 0, 0, 0, 0, 0, 0, 0, 0, 0];
            hhPotTempStoreTwo = [0, 0, 0, 0, 0, 0, 0, 0, 0, 0];
            hhPotSizes = [0, 0, 0, 0, 0, 0, 0, 0, 0];
            hhSmallestAllIn;

            for (i = 0; i < hhPotTotal.length; ++i) {
                hhPotTotal[i] += hhPotFlop[i];
            }

            for (i = 0; i < hhPotTotal.length; ++i) {
                if (hhChipsToBBConverter(hhPotTotal[i], hhDivisor, hhCurrency, hhInBBText) == hhStacks[i]) {
                    hhPotAllInCount++;
                    hhPotAllInStacks[i] = hhPotTotal[i];
                }
            }

            if (hhPotAllInCount == 0) {
                hhPotTurnOutput = hhPotTurnOutput.concat(hhCurrency, hhChipsToBBConverter(hhPotTotal.reduce(hhPotSum), hhDivisor, hhCurrency, hhInBBText), hhInBBText);
            }
            else if (hhPotAllInCount > 0) {
                for (i = 0; i < hhPotTempStore.length; ++i) {
                    hhPotTempStore[i] = hhPotTotal[i];
                }

                while (Math.max(...hhPotAllInStacks) > 0) {
                    hhSmallestAllIn = Math.max(...hhPotAllInStacks);

                    for (i = 0; i < hhPotAllInStacks.length; ++i) {
                        if (hhPotAllInStacks[i] > 0 && hhPotAllInStacks[i] < hhSmallestAllIn) {
                            hhSmallestAllIn = hhPotAllInStacks[i];
                        }
                    }

                    for (i = 0; i < hhPotTempStore.length; ++i) {
                        hhPotTempStoreTwo[i] = Math.min(hhPotTempStore[i], hhSmallestAllIn);
                    }


                    i = 0;
                    while (hhPotSizes[i] != 0) {
                        i++;
                    }

                    hhPotSizes[i] = hhPotTempStoreTwo.reduce(hhPotSum);


                    for (i = 0; i < hhPotAllInStacks.length; ++i) {
                        hhPotAllInStacks[i] = Math.max(0, hhPotAllInStacks[i] - hhSmallestAllIn);
                        hhPotTempStore[i] = Math.max(0, hhPotTempStore[i] - hhSmallestAllIn);
                    }
                }

                i = 0;
                while (hhPotSizes[i] != 0) {
                    ++i;
                }

                hhPotSizes[i] = hhPotTempStore.reduce(hhPotSum);

                hhPotTurnOutput = hhPotTurnOutput.concat("Main Pot: ", hhCurrency, hhChipsToBBConverter(hhPotSizes[0], hhDivisor, hhCurrency, hhInBBText), hhInBBText, ", ");

                i = 1;
                while (hhPotSizes[i] > 0.0001) {
                    hhPotTurnOutput = hhPotTurnOutput.concat("Side Pot ", i, ": ", hhCurrency, hhChipsToBBConverter(hhPotSizes[i], hhDivisor, hhCurrency, hhInBBText), hhInBBText, ", ");
                    ++i;
                }

                hhPotTurnOutput = hhPotTurnOutput.substring(0, hhPotTurnOutput.length - 2);

                if (hhPotTurnOutput.indexOf("Side Pot ") == -1) {
                    hhPotTurnOutput = hhPotTurnOutput.replace("Main Pot: ", "");
                }
            }


            hhPotAllInCount = 0;
            hhPotAllInStacks = [0, 0, 0, 0, 0, 0, 0, 0, 0, 0];
            hhPotTempStore = [0, 0, 0, 0, 0, 0, 0, 0, 0, 0];
            hhPotTempStoreTwo = [0, 0, 0, 0, 0, 0, 0, 0, 0, 0];
            hhPotSizes = [0, 0, 0, 0, 0, 0, 0, 0, 0];
            hhSmallestAllIn;

            for (i = 0; i < hhPotTotal.length; ++i) {
                hhPotTotal[i] += hhPotTurn[i];
            }

            for (i = 0; i < hhPotTotal.length; ++i) {
                if (hhChipsToBBConverter(hhPotTotal[i], hhDivisor, hhCurrency, hhInBBText) == hhStacks[i]) {
                    hhPotAllInCount++;
                    hhPotAllInStacks[i] = hhPotTotal[i];
                }
            }

            if (hhPotAllInCount == 0) {
                hhPotRiverOutput = hhPotRiverOutput.concat(hhCurrency, hhChipsToBBConverter(hhPotTotal.reduce(hhPotSum), hhDivisor, hhCurrency, hhInBBText), hhInBBText);
            }
            else if (hhPotAllInCount > 0) {
                for (i = 0; i < hhPotTempStore.length; ++i) {
                    hhPotTempStore[i] = hhPotTotal[i];
                }

                while (Math.max(...hhPotAllInStacks) > 0) {
                    hhSmallestAllIn = Math.max(...hhPotAllInStacks);

                    for (i = 0; i < hhPotAllInStacks.length; ++i) {
                        if (hhPotAllInStacks[i] > 0 && hhPotAllInStacks[i] < hhSmallestAllIn) {
                            hhSmallestAllIn = hhPotAllInStacks[i];
                        }
                    }

                    for (i = 0; i < hhPotTempStore.length; ++i) {
                        hhPotTempStoreTwo[i] = Math.min(hhPotTempStore[i], hhSmallestAllIn);
                    }


                    i = 0;
                    while (hhPotSizes[i] != 0) {
                        i++;
                    }

                    hhPotSizes[i] = hhPotTempStoreTwo.reduce(hhPotSum);


                    for (i = 0; i < hhPotAllInStacks.length; ++i) {
                        hhPotAllInStacks[i] = Math.max(0, hhPotAllInStacks[i] - hhSmallestAllIn);
                        hhPotTempStore[i] = Math.max(0, hhPotTempStore[i] - hhSmallestAllIn);
                    }
                }

                i = 0;
                while (hhPotSizes[i] > 0.0001) {
                    ++i;
                }

                hhPotSizes[i] = hhPotTempStore.reduce(hhPotSum);

                hhPotRiverOutput = hhPotRiverOutput.concat("Main Pot: ", hhCurrency, hhChipsToBBConverter(hhPotSizes[0], hhDivisor, hhCurrency, hhInBBText), hhInBBText, ", ");

                i = 1;
                while (hhPotSizes[i] != 0) {
                    hhPotRiverOutput = hhPotRiverOutput.concat("Side Pot ", i, ": ", hhCurrency, hhChipsToBBConverter(hhPotSizes[i], hhDivisor, hhCurrency, hhInBBText), hhInBBText, ", ");
                    ++i;
                }

                hhPotRiverOutput = hhPotRiverOutput.substring(0, hhPotRiverOutput.length - 2);

                if (hhPotRiverOutput.indexOf("Side Pot ") == -1) {
                    hhPotRiverOutput = hhPotRiverOutput.replace("Main Pot: ", "");
                }
            }


            hhPotAllInCount = 0;
            hhPotAllInStacks = [0, 0, 0, 0, 0, 0, 0, 0, 0, 0];
            hhPotTempStore = [0, 0, 0, 0, 0, 0, 0, 0, 0, 0];
            hhPotTempStoreTwo = [0, 0, 0, 0, 0, 0, 0, 0, 0, 0];
            hhPotSizes = [0, 0, 0, 0, 0, 0, 0, 0, 0];
            hhSmallestAllIn;

            for (i = 0; i < hhPotTotal.length; ++i) {
                hhPotTotal[i] += hhPotRiver[i];
            }

            for (i = 0; i < hhPotTotal.length; ++i) {
                if (hhChipsToBBConverter(hhPotTotal[i], hhDivisor, hhCurrency, hhInBBText) == hhStacks[i]) {
                    hhPotAllInCount++;
                    hhPotAllInStacks[i] = hhPotTotal[i];
                }
            }

            if (hhPotAllInCount == 0) {
                hhPotShowDownOutput = hhPotShowDownOutput.concat(hhCurrency, hhChipsToBBConverter(hhPotTotal.reduce(hhPotSum), hhDivisor, hhCurrency, hhInBBText), hhInBBText);
            }
            else if (hhPotAllInCount > 0) {
                for (i = 0; i < hhPotTempStore.length; ++i) {
                    hhPotTempStore[i] = hhPotTotal[i];
                }

                while (Math.max(...hhPotAllInStacks) > 0) {
                    hhSmallestAllIn = Math.max(...hhPotAllInStacks);

                    for (i = 0; i < hhPotAllInStacks.length; ++i) {
                        if (hhPotAllInStacks[i] > 0 && hhPotAllInStacks[i] < hhSmallestAllIn) {
                            hhSmallestAllIn = hhPotAllInStacks[i];
                        }
                    }

                    for (i = 0; i < hhPotTempStore.length; ++i) {
                        hhPotTempStoreTwo[i] = Math.min(hhPotTempStore[i], hhSmallestAllIn);
                    }


                    i = 0;
                    while (hhPotSizes[i] != 0) {
                        i++;
                    }

                    hhPotSizes[i] = hhPotTempStoreTwo.reduce(hhPotSum);


                    for (i = 0; i < hhPotAllInStacks.length; ++i) {
                        hhPotAllInStacks[i] = Math.max(0, hhPotAllInStacks[i] - hhSmallestAllIn);
                        hhPotTempStore[i] = Math.max(0, hhPotTempStore[i] - hhSmallestAllIn);
                    }
                }

                i = 0;
                while (hhPotSizes[i] != 0) {
                    ++i;
                }

                hhPotSizes[i] = hhPotTempStore.reduce(hhPotSum);

                hhPotShowDownOutput = hhPotShowDownOutput.concat("Main Pot: ", hhCurrency, hhChipsToBBConverter(hhPotSizes[0], hhDivisor, hhCurrency, hhInBBText), hhInBBText, ", ");

                i = 1;
                while (hhPotSizes[i] > 0.0001) {
                    hhPotShowDownOutput = hhPotShowDownOutput.concat("Side Pot ", i, ": ", hhCurrency, hhChipsToBBConverter(hhPotSizes[i], hhDivisor, hhCurrency, hhInBBText), hhInBBText, ", ");
                    ++i;
                }

                hhPotShowDownOutput = hhPotShowDownOutput.substring(0, hhPotShowDownOutput.length - 2);

                if (hhPotShowDownOutput.indexOf("Side Pot ") == -1) {
                    hhPotShowDownOutput = hhPotShowDownOutput.replace("Main Pot: ", "");
                }
            }

            return [hhPotFlopOutput, hhPotTurnOutput, hhPotRiverOutput, hhPotShowDownOutput];
        }


        function hhContainsFolds(hhText, hhNewLine, hhLineEnd, hhFoldCounter) {
            //Outputs when players have folded

            var hhTempNewLine, hhTempLineStart;
            var TempCheck = 0;

            if (hhNewLine.indexOf("[") == -1) {
                hhTempNewLine = hhText.substring(hhLineEnd, hhText.indexOf("\n", hhLineEnd) + 1);
                hhTempLineStart = hhText.indexOf("\n", hhLineEnd) + 1;

                while (TempCheck == 0) {
                    if (hhTempNewLine.indexOf("connected") == -1 && hhTempNewLine.indexOf("is sitting out") == -1 && hhTempNewLine.indexOf("has returned") == -1 && hhTempNewLine.indexOf("has timed out") == -1) {
                        TempCheck = 1;
                    }
                    else {
                        hhTempNewLine = hhText.substring(hhTempLineStart, hhText.indexOf("\n", hhTempLineStart) + 1);
                        hhTempLineStart = hhText.indexOf("\n", hhTempLineStart) + 1;
                    }
                }

                if (hhFoldCounter == 0) {
                    if (hhTempNewLine.indexOf("folds") != -1 && hhTempNewLine.indexOf("[") == -1) {
                        hhFoldCounter++;
                    }
                    else {
                        document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat(hhNewLine.substring(0, hhNewLine.lastIndexOf("folds") + 5).replace(":", ""), ", ");
                    }
                }
                else {
                    if (hhTempNewLine.indexOf("folds") != -1 && hhTempNewLine.indexOf("[") == -1) {
                        hhFoldCounter++;
                    }
                    else {
                        document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat(hhFoldCounter + 1, " players fold, ");
                        hhFoldCounter = 0;
                    }
                }
            }
            else {
                document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat(hhNewLine.substring(0, hhNewLine.indexOf("[")).replace(":", ""), "and shows ", hhCardConverter(hhNewLine.substring(hhNewLine.indexOf("[") + 1, hhNewLine.indexOf("]"))), ", ");
            }

            return hhFoldCounter;
        }


        function hhContainsCalls(hhText, hhNewLine, hhLineEnd, hhCallCounter) {
            //Outputs when players have called

            var TempCheck = 0;
            var hhTempNewLine = hhText.substring(hhLineEnd, hhText.indexOf("\n", hhLineEnd) + 1);
            var hhTempLineStart = hhText.indexOf("\n", hhLineEnd) + 1;

            while (TempCheck == 0) {
                if (hhTempNewLine.indexOf("connected") == -1 && hhTempNewLine.indexOf("is sitting out") == -1 && hhTempNewLine.indexOf("has returned") == -1 && hhTempNewLine.indexOf("has timed out") == -1) {
                    TempCheck = 1;
                }
                else {
                    hhTempNewLine = hhText.substring(hhTempLineStart, hhText.indexOf("\n", hhTempLineStart) + 1);
                    hhTempLineStart = hhText.indexOf("\n", hhTempLineStart) + 1;
                }
            }

            if (hhNewLine.indexOf("all-in") == -1 && hhTempNewLine.indexOf("all-in") == -1) {
                if (hhCallCounter == 0) {
                    if (hhTempNewLine.indexOf("calls") != -1) {
                        hhCallCounter++;
                    }
                    else {
                        document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat(hhNewLine.substring(0, hhNewLine.indexOf("calls") + 5).replace(":", ""), ", ");
                    }
                }
                else {
                    if (hhTempNewLine.indexOf("calls") != -1) {
                        hhCallCounter++;
                    }
                    else {
                        document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat(hhCallCounter + 1, " players call, ");
                        hhCallCounter = 0;
                    }
                }
            }
            else {
                if (hhNewLine.indexOf("all-in") == -1) {
                    document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat(hhNewLine.substring(0, hhNewLine.indexOf("calls") + 5).replace(":", ""), ", ");
                }
                else {
                    document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat(hhNewLine.substring(0, hhNewLine.length - 1).replace(":", ""), ", ");
                }
            }

            return hhCallCounter;
        }


        function hhContainsChecks(hhText, hhNewLine, hhLineEnd, hhCheckCounter) {
            //Outputs when players have checked

            var TempCheck = 0;
            var hhTempNewLine = hhText.substring(hhLineEnd, hhText.indexOf("\n", hhLineEnd) + 1);
            var hhTempLineStart = hhText.indexOf("\n", hhLineEnd) + 1;

            while (TempCheck == 0) {
                if (hhTempNewLine.indexOf("connected") == -1 && hhTempNewLine.indexOf("is sitting out") == -1 && hhTempNewLine.indexOf("has returned") == -1 && hhTempNewLine.indexOf("has timed out") == -1) {
                    TempCheck = 1;
                }
                else {
                    hhTempNewLine = hhText.substring(hhTempLineStart, hhText.indexOf("\n", hhTempLineStart) + 1);
                    hhTempLineStart = hhText.indexOf("\n", hhTempLineStart) + 1;
                }
            }

            if (hhCheckCounter == 0) {
                if (hhTempNewLine.indexOf("checks") != -1) {
                    hhCheckCounter++;
                }
                else {
                    document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat(hhNewLine.substring(0, hhNewLine.indexOf(" ")).replace(":", ""), " checks, ");
                }
            }
            else {
                if (hhTempNewLine.indexOf("checks") != -1) {
                    hhCheckCounter++;
                }
                else {
                    document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat(hhCheckCounter + 1, " players check, ");
                    hhCheckCounter = 0;
                }
            }

            return hhCheckCounter;
        }


        function hhContainsMucks(hhText, hhNewLine, hhLineEnd) {
            var hhTempLineEnd = hhLineEnd;
            var hhMuckedHand = "";
            hhTempNewLine = hhText.substring(hhTempLineEnd, hhText.indexOf("\n", hhTempLineEnd) + 1);

            while (hhTempLineEnd != 0) {
                if (hhTempNewLine.indexOf("mucked") != -1 && hhTempNewLine.indexOf(hhNewLine.substring(0, hhNewLine.indexOf(" ")).replace(":", "")) != -1) {
                    hhMuckedHand = hhCardConverter(hhTempNewLine.substring(hhTempNewLine.lastIndexOf("[") + 1, hhTempNewLine.lastIndexOf("]")));
                    hhTempLineEnd = 0;
                }
                else {
                    hhTempLineEnd = hhText.concat("\n").indexOf("\n", hhTempLineEnd) + 1;
                    hhTempNewLine = hhText.concat("\n").substring(hhTempLineEnd, hhText.indexOf("\n", hhTempLineEnd) + 1);
                }
            }

            document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat("\n> ", hhNewLine.substring(0, hhNewLine.indexOf(" ")).replace(":", ""), " mucked ", hhMuckedHand, "  ");
        }


        //This is where different types of hands are declared


        function hhTypeCommon(hhText, hhTypeNum, hhDivisor) {
            //Simplest type, represents NLHE, PLO, etc

            //Common terms
            var hhLineStart, hhNewLine, hhLineEnd, i;


            //Get basic hand info
            var hhBuyIn = hhGetBuyIn(hhText);
            var hhBlinds = hhGetBlinds(hhText);
            var hhButtonSeatNum = hhGetButtonPos(hhText);

            var hhPlayersInHand = [];
            var hhSeatsInHand = [];

            hhGetPlayersSeated(hhText, hhPlayersInHand, hhSeatsInHand, hhButtonSeatNum);


            //Get table positions
            var hhActualTP = [];

            hhGetTablePositions(hhText, hhSeatsInHand, hhActualTP, hhButtonSeatNum);


            //Clean data
            hhText = hhDeleteChatMsg(hhText);

            var hhHero = hhGetHeroName(hhText);
            var hhHeroPosTS = ["No Hero Pos Found"];
            hhText = hhReplacePlayerNames(hhText, hhActualTP, hhPlayersInHand, hhHero, hhHeroPosTS);

            if (hhHeroPosTS[0] == "No Hero Pos Found") {
                var hhHeroPos = "No Hero Pos Found";
            }
            else {
                var hhHeroPos = hhHeroPosTS[0];
            }


            //When Converted in BB, delete all occurances of currency labels
            if (hhDivisor == 0) {
                hhText = hhDeleteCurrencyLabels(hhText);
            }


            //Calculate Ante and Players Forced All In Due To Ante
            var hhAnteFAIPlayer = [];
            var hhAnteFAIStack = [];

            var TempStore = hhGetAnteAndFAIAnte(hhText, hhAnteFAIStack, hhAnteFAIPlayer, hhActualTP, hhAnte);

            var hhAnte = TempStore[0];
            var hhAnteCounter = TempStore[1];
            var hhAnteCounterFAI = TempStore[2];


            //Store stacks and bounties
            var hhStacks = [];
            var hhBounties = [];
            var hhSatOut = [];

            hhGetStacksAndBounties(hhText, hhStacks, hhBounties, hhSatOut, hhActualTP);


            //Calculate actual and theoretical blinds
            TempStore = [];
            TempStore = hhGetTheorAndActBlinds(hhText, hhBlinds);

            var hhActualBB = TempStore[0];
            var hhActualSB = TempStore[1];
            var hhTheorBB = TempStore[2];
            var hhTheorSB = TempStore[3];
            var hhBBFound = TempStore[4];
            var hhSBFound = TempStore[5];
            var hhBBCounter = TempStore[6];
            var hhCurrency = TempStore[7];
            var hhBringInString = TempStore[8];


            //Required containers to store pot values
            var hhPotTotal = [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]; //Store antes initially
            var hhPotPreflop = [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]; //All preflop action
            var hhPotFlop = [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]; //All flop action
            var hhPotTurn = [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]; //All turn action
            var hhPotRiver = [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]; //All river action

            var hhFoldTracker = [0, 0, 0, 0, 0, 0, 0, 0, 0, 0];//Tracks when players fold


            //Calculate Pot at every relevant point & time at which players fold
            hhPotCalculator(hhText, hhActualTP, hhStacks, hhAnteFAIPlayer, hhAnteFAIStack, hhAnte, hhPotTotal, hhPotPreflop, hhPotFlop, hhPotTurn, hhPotRiver, hhActualBB, hhActualSB, hhTheorBB, hhTheorSB, hhBBFound, hhSBFound, hhBBCounter, hhBringInString, hhFoldTracker, hhFoldTracker);


            //Calculate number of players at each stage of hand
            var hhPlayersFlop = hhPlayersInHand.length;
            var hhPlayersTurn = hhPlayersInHand.length;
            var hhPlayersRiver = hhPlayersInHand.length;

            for (i = 0; i < hhFoldTracker.length; ++i) {
                if (hhFoldTracker[i] == 1) {
                    hhPlayersFlop--;
                    hhPlayersTurn--;
                    hhPlayersRiver--;
                }
                else if (hhFoldTracker[i] == 2) {
                    hhPlayersTurn--;
                    hhPlayersRiver--;
                }
                else if (hhFoldTracker[i] == 3) {
                    hhPlayersRiver--;
                }
            }


            //Set Divisor and utilise it on Antes and Blinds
            var hhInBBText = "";

            if (hhDivisor == 0) {
                hhInBBText = " BB";
                hhCurrency = "";
                hhDivisor = Number(hhTheorBB);
            }

            hhTheorSB = hhChipsToBBConverter(hhTheorSB, hhDivisor, hhCurrency, hhInBBText);
            hhTheorBB = hhChipsToBBConverter(hhTheorBB, hhDivisor, hhCurrency, hhInBBText);
            hhActualSB = hhChipsToBBConverter(hhActualSB, hhDivisor, hhCurrency, hhInBBText);
            hhActualBB = hhChipsToBBConverter(hhActualBB, hhDivisor, hhCurrency, hhInBBText);
            hhAnte = hhChipsToBBConverter(hhAnte, hhDivisor, hhCurrency, hhInBBText);

            for (i = 0; i < hhStacks.length; ++i) {
                hhStacks[i] = hhChipsToBBConverter(hhStacks[i], hhDivisor, hhCurrency, hhInBBText);
            }

            for (i = 0; i < hhAnteFAIStack.length; ++i) {
                hhAnteFAIStack[i] = hhChipsToBBConverter(hhAnteFAIStack[i], hhDivisor, hhCurrency, hhInBBText);
            }


            //Calculate Pots to be output
            TempStore = [];
            TempStore = hhPotSummaryOutput(hhPotTotal, hhPotPreflop, hhPotFlop, hhPotTurn, hhPotRiver, hhStacks, hhCurrency, hhDivisor, hhInBBText);

            var hhPotFlopOutput = TempStore[0];
            var hhPotTurnOutput = TempStore[1];
            var hhPotRiverOutput = TempStore[2];
            var hhPotShowDownOutput = TempStore[3];


            //Finds the rake taken in the hand
            var hhRake = hhGetRake(hhText, hhDivisor, hhCurrency, hhInBBText);
            hhPotShowDownOutput = hhPotShowDownOutput.concat(" | Rake: ", hhCurrency, hhRake, hhInBBText);


            //Beginning of output
            var hhTypesOfGames = ["Hold'em No Limit", "Hold'em Pot Limit", "Omaha No Limit", "Omaha Pot Limit", "Omaha Hi/Lo No Limit", "Omaha Hi/Lo Pot Limit", "5 Card Omaha No Limit", "5 Card Omaha Pot Limit", "5 Card Omaha Hi/Lo Pot Limit", "6 Card Omaha Pot Limit", "Swap Hold'em No Limit", "Swap Hold'em Pot Limit", "Fusion No Limit", "Fusion Pot Limit", "Showtime Hold'em No Limit", "Showtime Hold'em Pot Limit"];
            var hhTypeName = hhTypesOfGames[hhTypeNum];


            //Output beginning of hh
            hhOutputIntro(hhTypeName, hhAnteCounter, hhCurrency, hhTheorSB, hhInBBText, hhTheorBB, hhInBBText, hhAnte, hhAnteFAIStack, hhBuyIn);


            //Normalize starting positions and output stacks/bounties/sit outs
            hhNormalizeOutputStacks(hhActualTP, hhStacks, hhBounties, hhCurrency, hhInBBText, hhSatOut, hhHeroPos);


            //Displays posted blinds and antes
            hhOutputBlindsAntes(hhAnteCounter, hhAnteCounterFAI, hhAnteFAIPlayer, hhActualTP, hhCurrency, hhAnteFAIStack, hhInBBText, hhAnte, hhBBCounter, hhTheorBB, hhSBFound, hhActualSB, hhTheorSB, hhBBFound, hhActualBB, hhBringInString, hhHeroPos);


            //Divide neccessary data by hhDivisor so BB conversion elligible
            if (hhInBBText == " BB") {
                hhText = hhBBConverter(hhText, hhDivisor, hhCurrency, hhInBBText);
            }


            //Displays actual hand action
            var hhFoldCounter = 0;
            var hhCallCounter = 0;
            var hhCheckCounter = 0;
            var hhShowdown = 0;
            var hhFlop, hhTurn, hhRiver, hhTempNewLine;

            hhLineStart = 0;
            hhLineEnd = hhText.indexOf("\n") + 1;

            while (hhLineEnd != 0) {
                hhNewLine = hhText.substring(hhLineStart, hhLineEnd);

                if (hhNewLine.indexOf("Dealt to") != -1) {
                    document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat(hhNewLine.substring(0, hhNewLine.indexOf("[") - 1), ":  ", hhCardConverter(hhNewLine.substring(hhNewLine.indexOf("[") + 1, hhNewLine.lastIndexOf("]"))), "\n> ");
                }
                else if (hhNewLine.indexOf("folds") != -1) {
                    hhFoldCounter = hhContainsFolds(hhText, hhNewLine, hhLineEnd, hhFoldCounter);
                }
                else if (hhNewLine.indexOf("calls") != -1) {
                    hhCallCounter = hhContainsCalls(hhText, hhNewLine, hhLineEnd, hhCallCounter);
                }
                else if (hhNewLine.indexOf("raises") != -1) {
                    document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat(hhNewLine.substring(0, hhNewLine.indexOf("raises") + 6).replace(":", ""), hhNewLine.substring(hhNewLine.lastIndexOf("to") - 1, hhNewLine.length - 1), ", ");
                }
                else if (hhNewLine.indexOf("bets") != -1) {
                    document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat(hhNewLine.substring(0, hhNewLine.length - 1).replace(":", ""), ", ");
                }
                else if (hhNewLine.indexOf("checks") != -1) {
                    hhCheckCounter = hhContainsChecks(hhText, hhNewLine, hhLineEnd, hhCheckCounter);
                }
                else if (hhNewLine.indexOf("Uncalled bet") != -1) {
                    document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat(hhNewLine.substring(0, hhNewLine.length - 1).replace(":", ""), "  ");
                }
                else if (hhNewLine.indexOf("FLOP") != -1) {
                    document.getElementById('hhInput').value = document.getElementById('hhInput').value.substring(0, document.getElementById('hhInput').value.length - 2);
                    hhFlop = hhCardConverter(hhNewLine.substring(hhNewLine.lastIndexOf("[") + 1, hhNewLine.lastIndexOf("]")));
                    document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat("\n\n> **FLOP**  (", hhPotFlopOutput, ") ", hhPlayersFlop, " players\n> ", hhFlop, "\n> ");
                }
                else if (hhNewLine.indexOf("TURN") != -1) {
                    document.getElementById('hhInput').value = document.getElementById('hhInput').value.substring(0, document.getElementById('hhInput').value.length - 2);
                    hhTurn = hhCardConverter(hhNewLine.substring(hhNewLine.lastIndexOf("[") + 1, hhNewLine.lastIndexOf("]")));
                    document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat("\n\n> **TURN**  (", hhPotTurnOutput, ") ", hhPlayersTurn, " players\n> ", hhFlop, " ", hhTurn, "\n> ");
                }
                else if (hhNewLine.indexOf("RIVER") != -1) {
                    document.getElementById('hhInput').value = document.getElementById('hhInput').value.substring(0, document.getElementById('hhInput').value.length - 2);
                    hhRiver = hhCardConverter(hhNewLine.substring(hhNewLine.lastIndexOf("[") + 1, hhNewLine.lastIndexOf("]")));
                    document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat("\n\n> **RIVER**  (", hhPotRiverOutput, ") ", hhPlayersRiver, " players\n> ", hhFlop, " ", hhTurn, " ", hhRiver, "\n> ");
                }
                else if (hhNewLine.indexOf("SHOW DOWN") != -1) {
                    hhShowdown = 1;
                    document.getElementById('hhInput').value = document.getElementById('hhInput').value.substring(0, document.getElementById('hhInput').value.length - 2);
                    document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat("\n\n> **SHOW DOWN** (", hhPotShowDownOutput, ") ||");
                }
                else if (hhNewLine.indexOf("collected") != -1 && hhNewLine.indexOf("Seat ") == -1) {
                    document.getElementById('hhInput').value = document.getElementById('hhInput').value.substring(0, document.getElementById('hhInput').value.length - 2);
                    document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat("\n> ", hhNewLine.substring(0, hhNewLine.length - 1), "  ");
                }
                else if (hhNewLine.indexOf("cashed out ") != -1) {
                    document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat("\n> ", hhNewLine.substring(0, hhNewLine.indexOf(" ")), " cashed out :money_with_wings: ", hhNewLine.substring(hhNewLine.indexOf("for"), hhNewLine.length - 1), "  ");
                }
                else if (hhNewLine.indexOf("eliminating") != -1) {
                    if (hhNewLine.indexOf("increases") != -1) {
                        document.getElementById('hhInput').value = document.getElementById('hhInput').value.substring(0, document.getElementById('hhInput').value.length - 2);
                        document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat("\n> ", hhNewLine.substring(0, hhNewLine.indexOf("bounty")), ":dart: increases", hhNewLine.substring(hhNewLine.lastIndexOf("to") - 1, hhNewLine.length - 1), "  ");
                    }
                    else {
                        document.getElementById('hhInput').value = document.getElementById('hhInput').value.substring(0, document.getElementById('hhInput').value.length - 2);
                        document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat("\n> ", hhNewLine.substring(0, hhNewLine.length - 1), "  ");
                    }
                }
                else if (hhNewLine.indexOf("wins the tournament") != -1) {
                    document.getElementById('hhInput').value = document.getElementById('hhInput').value.substring(0, document.getElementById('hhInput').value.length - 2);
                    document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat("\n> ", hhNewLine.substring(0, hhNewLine.indexOf(" -")), " :trophy: :champagne: ");
                }
                else if (hhNewLine.indexOf("shows") != -1) {
                    if (hhNewLine.indexOf("(") != -1) {
                        document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat("\n> ", hhNewLine.substring(0, hhNewLine.indexOf("[")).replace(":", ""), hhCardConverter(hhNewLine.substring(hhNewLine.lastIndexOf("[") + 1, hhNewLine.lastIndexOf("]"))), hhNewLine.substring(hhNewLine.indexOf("]") + 1, hhNewLine.length - 1), "  ");
                    }
                    else {
                        document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat("\n> ", hhNewLine.substring(0, hhNewLine.indexOf("[")).replace(":", ""), hhCardConverter(hhNewLine.substring(hhNewLine.lastIndexOf("[") + 1, hhNewLine.lastIndexOf("]"))), "  ");
                    }
                }
                else if (hhNewLine.indexOf("mucks") != -1) {
                    hhContainsMucks(hhText, hhNewLine, hhLineEnd);
                }
                else if (hhNewLine.indexOf("swaps") != -1) {
                    if (hhNewLine.indexOf("[") != -1) {
                        document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat(hhNewLine.substring(0, hhNewLine.indexOf("[")).replace(":", ""), hhCardConverter(hhNewLine.substring(hhNewLine.lastIndexOf("[") + 1, hhNewLine.lastIndexOf("]"))), ", ");
                    }
                    else {
                        document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat(hhNewLine.substring(0, hhNewLine.length - 1).replace(":", ""), ", ");
                    }
                }

                hhLineStart = hhLineEnd;
                hhLineEnd = hhText.indexOf("\n", hhLineEnd) + 1;
            }

            if (hhShowdown == 1) {
                document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat("||");
            }

            return;
        }


        function hhTypeFixedLimit(hhText, hhTypeNum, hhDivisor) {
            //Simplest type, represents NLHE, PLO, etc

            //Common terms
            var hhLineStart, hhNewLine, hhLineEnd, i;


            //Get basic hand info
            var hhBuyIn = hhGetBuyIn(hhText);
            var hhBlinds = hhGetBlinds(hhText);
            var hhButtonSeatNum = hhGetButtonPos(hhText);

            var hhPlayersInHand = [];
            var hhSeatsInHand = [];

            hhGetPlayersSeated(hhText, hhPlayersInHand, hhSeatsInHand, hhButtonSeatNum);


            //Get table positions
            var hhActualTP = [];

            hhGetTablePositions(hhText, hhSeatsInHand, hhActualTP, hhButtonSeatNum);


            //Clean data
            hhText = hhDeleteChatMsg(hhText);

            var hhHero = hhGetHeroName(hhText);
            var hhHeroPosTS = ["No Hero Pos Found"];
            hhText = hhReplacePlayerNames(hhText, hhActualTP, hhPlayersInHand, hhHero, hhHeroPosTS);

            if (hhHeroPosTS[0] == "No Hero Pos Found") {
                var hhHeroPos = "No Hero Pos Found";
            }
            else {
                var hhHeroPos = hhHeroPosTS[0];
            }


            //When Converted in BB, delete all occurances of currency labels
            if (hhDivisor == 0) {
                hhText = hhDeleteCurrencyLabels(hhText);
            }


            //Calculate Ante and Players Forced All In Due To Ante
            var hhAnteFAIPlayer = [];
            var hhAnteFAIStack = [];

            var TempStore = hhGetAnteAndFAIAnte(hhText, hhAnteFAIStack, hhAnteFAIPlayer, hhActualTP, hhAnte);

            var hhAnte = TempStore[0];
            var hhAnteCounter = TempStore[1];
            var hhAnteCounterFAI = TempStore[2];


            //Store stacks and bounties
            var hhStacks = [];
            var hhBounties = [];
            var hhSatOut = [];

            hhGetStacksAndBounties(hhText, hhStacks, hhBounties, hhSatOut, hhActualTP);


            //Calculate actual and theoretical blinds
            TempStore = [];
            TempStore = hhGetTheorAndActBlinds(hhText, hhBlinds);

            var hhActualBB = TempStore[0];
            var hhActualSB = TempStore[1];
            var hhTheorBB = TempStore[2] / 2;
            var hhTheorSB = TempStore[3] / 2;
            var hhBBFound = TempStore[4];
            var hhSBFound = TempStore[5];
            var hhBBCounter = TempStore[6];
            var hhCurrency = TempStore[7];
            var hhBringInString = TempStore[8];


            //Required containers to store pot values
            var hhPotTotal = [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]; //Store antes initially
            var hhPotPreflop = [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]; //All preflop action
            var hhPotFlop = [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]; //All flop action
            var hhPotTurn = [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]; //All turn action
            var hhPotRiver = [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]; //All river action

            var hhFoldTracker = [0, 0, 0, 0, 0, 0, 0, 0, 0, 0];//Tracks when players fold


            //Calculate Pot at every relevant point & time at which players fold
            hhPotCalculator(hhText, hhActualTP, hhStacks, hhAnteFAIPlayer, hhAnteFAIStack, hhAnte, hhPotTotal, hhPotPreflop, hhPotFlop, hhPotTurn, hhPotRiver, hhActualBB, hhActualSB, hhTheorBB, hhTheorSB, hhBBFound, hhSBFound, hhBBCounter, hhBringInString, hhFoldTracker, hhFoldTracker);


            //Calculate number of players at each stage of hand
            var hhPlayersFlop = hhPlayersInHand.length;
            var hhPlayersTurn = hhPlayersInHand.length;
            var hhPlayersRiver = hhPlayersInHand.length;

            for (i = 0; i < hhFoldTracker.length; ++i) {
                if (hhFoldTracker[i] == 1) {
                    hhPlayersFlop--;
                    hhPlayersTurn--;
                    hhPlayersRiver--;
                }
                else if (hhFoldTracker[i] == 2) {
                    hhPlayersTurn--;
                    hhPlayersRiver--;
                }
                else if (hhFoldTracker[i] == 3) {
                    hhPlayersRiver--;
                }
            }


            //Set Divisor and utilise it on Antes and Blinds
            var hhInBBText = "";
            var hhLimitMultiplier = 2;

            if (hhDivisor == 0) {
                hhInBBText = " BB";
                hhCurrency = "";
                hhLimitMultiplier = 1;
                hhDivisor = Number(hhTheorBB);
            }

            hhTheorSB = hhChipsToBBConverter(hhTheorSB, hhDivisor, hhCurrency, hhInBBText);
            hhTheorBB = hhChipsToBBConverter(hhTheorBB, hhDivisor, hhCurrency, hhInBBText);
            hhActualSB = hhChipsToBBConverter(hhActualSB, hhDivisor, hhCurrency, hhInBBText);
            hhActualBB = hhChipsToBBConverter(hhActualBB, hhDivisor, hhCurrency, hhInBBText);
            hhAnte = hhChipsToBBConverter(hhAnte, hhDivisor, hhCurrency, hhInBBText);

            for (i = 0; i < hhStacks.length; ++i) {
                hhStacks[i] = hhChipsToBBConverter(hhStacks[i], hhDivisor, hhCurrency, hhInBBText);
            }

            for (i = 0; i < hhAnteFAIStack.length; ++i) {
                hhAnteFAIStack[i] = hhChipsToBBConverter(hhAnteFAIStack[i], hhDivisor, hhCurrency, hhInBBText);
            }


            //Calculate Pots to be output
            TempStore = [];
            TempStore = hhPotSummaryOutput(hhPotTotal, hhPotPreflop, hhPotFlop, hhPotTurn, hhPotRiver, hhStacks, hhCurrency, hhDivisor, hhInBBText);

            var hhPotFlopOutput = TempStore[0];
            var hhPotTurnOutput = TempStore[1];
            var hhPotRiverOutput = TempStore[2];
            var hhPotShowDownOutput = TempStore[3];


            //Finds the rake taken in the hand
            var hhRake = hhGetRake(hhText, hhDivisor, hhCurrency, hhInBBText);
            hhPotShowDownOutput = hhPotShowDownOutput.concat(" | Rake: ", hhCurrency, hhRake, hhInBBText);


            //Beginning of output
            var hhTypesOfGames = ["Hold'em Limit", "Omaha Limit", "Omaha Hi/Lo Limit", "Showtime Hold'em Limit"];
            var hhTypeName = hhTypesOfGames[hhTypeNum];


            //Output beginning of hh
            if (hhBuyIn != "") {
                document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat("> Buy In: ", hhBuyIn, "\n");
            }

            document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat("> Game Type: ", hhTypeName, "\n", "> Ante: ");

            if (hhAnteCounter == 0) {
                document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat("None | Limits: ", hhCurrency, hhTheorSB * hhLimitMultiplier, hhInBBText, " / ", hhCurrency, hhTheorBB * hhLimitMultiplier, hhInBBText, "\n\n> Stacks:\n");
            }
            else if (hhAnte == 0) {
                document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat(hhCurrency, Math.max.apply(Math, hhAnteFAIStack), hhInBBText, " | Limits: ", hhCurrency, hhTheorSB * hhLimitMultiplier, hhInBBText, " / ", hhCurrency, hhTheorBB * hhLimitMultiplier, hhInBBText, "\n\n> Stacks:\n");
            }
            else {
                document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat(hhCurrency, hhAnte, hhInBBText, " | Limits: ", hhCurrency, hhTheorSB * hhLimitMultiplier, hhInBBText, " / ", hhCurrency, hhTheorBB * hhLimitMultiplier, hhInBBText, "\n\n> Stacks:\n");
            }


            //Normalize starting positions and output stacks/bounties/sit outs
            hhNormalizeOutputStacks(hhActualTP, hhStacks, hhBounties, hhCurrency, hhInBBText, hhSatOut, hhHeroPos);


            //Displays posted blinds and antes
            hhOutputBlindsAntes(hhAnteCounter, hhAnteCounterFAI, hhAnteFAIPlayer, hhActualTP, hhCurrency, hhAnteFAIStack, hhInBBText, hhAnte, hhBBCounter, hhTheorBB, hhSBFound, hhActualSB, hhTheorSB, hhBBFound, hhActualBB, hhBringInString, hhHeroPos);


            //Divide neccessary data by hhDivisor so BB conversion elligible
            if (hhInBBText == " BB") {
                hhText = hhBBConverter(hhText, hhDivisor, hhCurrency, hhInBBText);
            }


            //Displays actual hand action
            var hhFoldCounter = 0;
            var hhCallCounter = 0;
            var hhCheckCounter = 0;
            var hhShowdown = 0;
            var hhFlop, hhTurn, hhRiver, hhTempNewLine;

            hhLineStart = 0;
            hhLineEnd = hhText.indexOf("\n") + 1;

            while (hhLineEnd != 0) {
                hhNewLine = hhText.substring(hhLineStart, hhLineEnd);

                if (hhNewLine.indexOf("Dealt to") != -1) {
                    document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat(hhNewLine.substring(0, hhNewLine.indexOf("[") - 1), ":  ", hhCardConverter(hhNewLine.substring(hhNewLine.indexOf("[") + 1, hhNewLine.lastIndexOf("]"))), "\n> ");
                }
                else if (hhNewLine.indexOf("folds") != -1) {
                    hhFoldCounter = hhContainsFolds(hhText, hhNewLine, hhLineEnd, hhFoldCounter);
                }
                else if (hhNewLine.indexOf("calls") != -1) {
                    hhCallCounter = hhContainsCalls(hhText, hhNewLine, hhLineEnd, hhCallCounter);
                }
                else if (hhNewLine.indexOf("raises") != -1) {
                    document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat(hhNewLine.substring(0, hhNewLine.indexOf("raises") + 6).replace(":", ""), hhNewLine.substring(hhNewLine.lastIndexOf("to") - 1, hhNewLine.length - 1), ", ");
                }
                else if (hhNewLine.indexOf("bets") != -1) {
                    document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat(hhNewLine.substring(0, hhNewLine.length - 1).replace(":", ""), ", ");
                }
                else if (hhNewLine.indexOf("checks") != -1) {
                    hhCheckCounter = hhContainsChecks(hhText, hhNewLine, hhLineEnd, hhCheckCounter);
                }
                else if (hhNewLine.indexOf("Uncalled bet") != -1) {
                    document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat(hhNewLine.substring(0, hhNewLine.length - 1).replace(":", ""), "  ");
                }
                else if (hhNewLine.indexOf("FLOP") != -1) {
                    document.getElementById('hhInput').value = document.getElementById('hhInput').value.substring(0, document.getElementById('hhInput').value.length - 2);
                    hhFlop = hhCardConverter(hhNewLine.substring(hhNewLine.lastIndexOf("[") + 1, hhNewLine.lastIndexOf("]")));
                    document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat("\n\n> **FLOP**  (", hhPotFlopOutput, ") ", hhPlayersFlop, " players\n> ", hhFlop, "\n> ");
                }
                else if (hhNewLine.indexOf("TURN") != -1) {
                    document.getElementById('hhInput').value = document.getElementById('hhInput').value.substring(0, document.getElementById('hhInput').value.length - 2);
                    hhTurn = hhCardConverter(hhNewLine.substring(hhNewLine.lastIndexOf("[") + 1, hhNewLine.lastIndexOf("]")));
                    document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat("\n\n> **TURN**  (", hhPotTurnOutput, ") ", hhPlayersTurn, " players\n> ", hhFlop, " ", hhTurn, "\n> ");
                }
                else if (hhNewLine.indexOf("RIVER") != -1) {
                    document.getElementById('hhInput').value = document.getElementById('hhInput').value.substring(0, document.getElementById('hhInput').value.length - 2);
                    hhRiver = hhCardConverter(hhNewLine.substring(hhNewLine.lastIndexOf("[") + 1, hhNewLine.lastIndexOf("]")));
                    document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat("\n\n> **RIVER**  (", hhPotRiverOutput, ") ", hhPlayersRiver, " players\n> ", hhFlop, " ", hhTurn, " ", hhRiver, "\n> ");
                }
                else if (hhNewLine.indexOf("SHOW DOWN") != -1) {
                    hhShowdown = 1;
                    document.getElementById('hhInput').value = document.getElementById('hhInput').value.substring(0, document.getElementById('hhInput').value.length - 2);
                    document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat("\n\n> **SHOW DOWN** (", hhPotShowDownOutput, ") ||");
                }
                else if (hhNewLine.indexOf("collected") != -1 && hhNewLine.indexOf("Seat ") == -1) {
                    document.getElementById('hhInput').value = document.getElementById('hhInput').value.substring(0, document.getElementById('hhInput').value.length - 2);
                    document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat("\n> ", hhNewLine.substring(0, hhNewLine.length - 1), "  ");
                }
                else if (hhNewLine.indexOf("cashed out ") != -1) {
                    document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat("\n> ", hhNewLine.substring(0, hhNewLine.indexOf(" ")), " cashed out :money_with_wings: ", hhNewLine.substring(hhNewLine.indexOf("for"), hhNewLine.length - 1), "  ");
                }
                else if (hhNewLine.indexOf("eliminating") != -1) {
                    if (hhNewLine.indexOf("increases") != -1) {
                        document.getElementById('hhInput').value = document.getElementById('hhInput').value.substring(0, document.getElementById('hhInput').value.length - 2);
                        document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat("\n> ", hhNewLine.substring(0, hhNewLine.indexOf("bounty")), ":dart: increases", hhNewLine.substring(hhNewLine.lastIndexOf("to") - 1, hhNewLine.length - 1), "  ");
                    }
                    else {
                        document.getElementById('hhInput').value = document.getElementById('hhInput').value.substring(0, document.getElementById('hhInput').value.length - 2);
                        document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat("\n> ", hhNewLine.substring(0, hhNewLine.length - 1), "  ");
                    }
                }
                else if (hhNewLine.indexOf("wins the tournament") != -1) {
                    document.getElementById('hhInput').value = document.getElementById('hhInput').value.substring(0, document.getElementById('hhInput').value.length - 2);
                    document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat("\n> ", hhNewLine.substring(0, hhNewLine.indexOf(" -")), " :trophy: :champagne: ");
                }
                else if (hhNewLine.indexOf("shows") != -1) {
                    if (hhNewLine.indexOf("(") != -1) {
                        document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat("\n> ", hhNewLine.substring(0, hhNewLine.indexOf("[")).replace(":", ""), hhCardConverter(hhNewLine.substring(hhNewLine.lastIndexOf("[") + 1, hhNewLine.lastIndexOf("]"))), hhNewLine.substring(hhNewLine.indexOf("]") + 1, hhNewLine.length - 1), "  ");
                    }
                    else {
                        document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat("\n> ", hhNewLine.substring(0, hhNewLine.indexOf("[")).replace(":", ""), hhCardConverter(hhNewLine.substring(hhNewLine.lastIndexOf("[") + 1, hhNewLine.lastIndexOf("]"))), "  ");
                    }
                }
                else if (hhNewLine.indexOf("mucks") != -1) {
                    hhContainsMucks(hhText, hhNewLine, hhLineEnd);
                }
                else if (hhNewLine.indexOf("swaps") != -1) {
                    if (hhNewLine.indexOf("[") != -1) {
                        document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat(hhNewLine.substring(0, hhNewLine.indexOf("[")).replace(":", ""), hhCardConverter(hhNewLine.substring(hhNewLine.lastIndexOf("[") + 1, hhNewLine.lastIndexOf("]"))), ", ");
                    }
                    else {
                        document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat(hhNewLine.substring(0, hhNewLine.length - 1).replace(":", ""), ", ");
                    }
                }

                hhLineStart = hhLineEnd;
                hhLineEnd = hhText.indexOf("\n", hhLineEnd) + 1;
            }

            if (hhShowdown == 1) {
                document.getElementById('hhInput').value = document.getElementById('hhInput').value.concat("||");
            }

            return;
        }

    </script>

    <p class="title">
        Welcome to the Temporary Home for the Discord Hand History Converter! <br>
    </p>

    <textarea class="textarea" name="hhInput" id="hhInput" placeholder="Copy and paste your hand history here, then press 'Convert Hand History' to get your converted hand history!"></textarea><br>
    <button class="button" id="submit" onclick="hhConvert()">Convert Hand History</button>
    <div id="copy-wrapper">
        <button id="copy" class="button">Copy All</button>
        <div id="copied-notification">Copied</div>
        <button class="button" id="reset" onclick="hhReset()">Reset</button>
    </div>

    Convert in Chips
    <label class="switch">
        <input type="checkbox" onclick="hhToggle()">
        <span class="slider round"></span>
    </label>
    Convert in Big Blinds<br><br>

    Currently Accepted Game Types:
    <ul class="disclaimer">
        <li>Hold'em No Limit </li>
        <li>Hold'em Pot Limit </li>
        <li>Hold'em Limit </li>
        <li>Omaha No Limit </li>
        <li>Omaha Pot Limit </li>
        <li>Omaha Limit </li>
        <li>Omaha Hi/Lo No Limit </li>
        <li>Omaha Hi/Lo Pot Limit </li>
        <li>Omaha Hi/Lo Limit </li>
        <li>5 Card Omaha No Limit </li>
        <li>5 Card Omaha Pot Limit </li>
        <li>5 Card Omaha Hi/Lo Pot Limit </li>
        <li>6 Card Omaha Pot Limit </li>
        <li>Swap Hold'em No Limit </li>
        <li>Swap Hold'em Pot Limit </li>
        <li>Fusion No Limit </li>
        <li>Fusion Pot Limit </li>
        <li>Showtime Hold'em No Limit </li>
        <li>Showtime Hold'em Pot Limit </li>
        <li>Showtime Hold'em Limit </li>
    </ul>

    <script>
        const copyButton = document.getElementById('copy');

        const lastTimeout = null;

        copyButton.addEventListener('click', () => {
            hhCopy();

            const notification = document.getElementById('copied-notification');
            notification.classList.add('visible');
            setTimeout(() => {
                notification.classList.add('animating');
            }, 100);

            setTimeout(() => {
                notification.classList.remove('visible', 'animating');
            }, 1100);
        })
    </script>
</body>
</html>
