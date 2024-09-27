// Function to apply (or remove) XP to selected actors
async function applyXP(selectedActors, xpAmount) {
    let perPlayerXP = Math.floor(xpAmount / selectedActors.length);
    for (let actor of selectedActors) {
        let currentXP = actor.system.details.xp.value;  // Updated for v10
        let newXP = currentXP + perPlayerXP;
        newXP = Math.max(newXP, 0);  // Prevent XP from going below zero
        await actor.update({"system.details.xp.value": newXP});  // Updated for v10
        console.log(`${actor.name} now has ${newXP} XP.`);
    }
    ui.notifications.info(`Applied ${perPlayerXP} XP to ${selectedActors.length} characters.`);
}

// Save checked player selections
function saveSelectedOptions(selectedActorIds) {
    localStorage.setItem("lastSelectedPlayers", JSON.stringify(selectedActorIds));
}

// Load checked player selections
function loadSelectedOptions() {
    return JSON.parse(localStorage.getItem("lastSelectedPlayers") || "[]");
}

// Function to update XP values based on input changes or player selection changes
function updateXPFields(html) {
    let partyXPInput = html.find("#party-xp-input");
    let playerXPInput = html.find("#player-xp-input");
    let selectedPlayers = html.find("input[name=selectedPlayer]:checked").length || 1;

    // When party XP is changed, update player XP
    partyXPInput.on('input', function () {
        let partyXP = parseInt(partyXPInput.val()) || 0;
        playerXPInput.val(Math.floor(partyXP / selectedPlayers));
    });

    // When player XP is changed, update party XP
    playerXPInput.on('input', function () {
        let playerXP = parseInt(playerXPInput.val()) || 0;
        partyXPInput.val(playerXP * selectedPlayers);
    });

    // Recalculate XP fields when player selection changes
    html.find("input[name=selectedPlayer]").on('change', function () {
        selectedPlayers = html.find("input[name=selectedPlayer]:checked").length || 1;
        let partyXP = parseInt(partyXPInput.val()) || 0;
        playerXPInput.val(Math.floor(partyXP / selectedPlayers));
    });
}

// Create a dialog to select players and input XP
let content = `<p><b>Distribute XP</b></p>
<p><label for="party-xp-input">Party XP:</label> <input type="number" id="party-xp-input" value="0"></p>
<p><label for="player-xp-input">Per Player XP:</label> <input type="number" id="player-xp-input" value="0"></p>
<p>Select the players to distribute XP to:</p>
<form id="player-form">`;

// Load previously selected players
let lastSelected = loadSelectedOptions();

game.actors.forEach(actor => {
    if (actor.hasPlayerOwner) {
        let isChecked = lastSelected.includes(actor.id) ? "checked" : "";
        content += `<p><input type="checkbox" name="selectedPlayer" value="${actor.id}" ${isChecked}> ${actor.name}</p>`;
    }
});

content += `</form>`;

new Dialog({
    title: "Distribute XP",
    content: content,
    render: html => {
        updateXPFields(html);  // Initialize field updates
    },
    buttons: {
        apply: {
            label: "Apply",
            callback: async (html) => {
                let xpValue = parseInt(html.find("#party-xp-input").val());
                let selectedActorIds = Array.from(html.find("input[name=selectedPlayer]:checked")).map(el => el.value);
                let selectedActors = selectedActorIds.map(id => game.actors.get(id));

                // Save the selected options for next time
                saveSelectedOptions(selectedActorIds);

                if (selectedActors.length > 0 && !isNaN(xpValue)) {
                    applyXP(selectedActors, xpValue);
                } else {
                    ui.notifications.warn("Please enter a valid XP value and select at least one player.");
                }
            }
        }
    }
}).render(true);
