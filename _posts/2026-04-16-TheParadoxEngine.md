---
layout: post
draft: false
title: "The Paradox Engine"
description: 'A randomization tool I use to mix up my somatic practices.'
category: self
tags: [ proactive-prep, somatic work, parasympathetic, hrv, chaos-engineering ]
---
{::options parse_block_html="true" /}

## The Tool

The basic somatic process is: Shake/Hum/Abide.

One Paradox: Roll one D10, select the verb. Roll one D12, select Context.
Gary Larson: Roll 8 pairs of paradoxes (4 pairs). Optional, switch the order of verb/context
Program: Roll odd/even, starting with verbs (odd) or context (even), roll values until it seems you have enough
Pool: Roll one d12, divide by 4 (1 -4 becomes 1, 5-8 becomes 2, etc.) Roll that many verbs. Same thing with Context. 

| #  | Verb (d10) | #  | Context (d12) |
|----|------------|----|---------------|
| 1  | Shake      | 1  | Self          |
| 2  | Breath     | 2  | Engine        |
| 3  | Abide      | 3  | Liquid        |
| 4  | Stabilize  | 4  | Firmware      |
| 5  | Expand     | 5  | Mu (Null)     |
| 6  | Coordinate | 6  | Context       |
| 7  | Engage     | 7  | Other         |
| 8  | Inhabit    | 8  | Network       |
| 9  | Withdraw   | 9  | Transact      |
| 10 | Mu (Null)  | 10 | Egregore      |
| -- | --         | 11 | Xenobiotic    |
| -- | --         | 12 | Terminate     |

## Try It Out

{::nomarkdown}
<!-- Section 1: Single Paradox -->
<div style="margin: 20px 0; padding: 20px; border: 2px solid #333; border-radius: 8px; background: #f5f5f5;">
  <h3 style="margin-top: 0; color: #333;">Section 1: Single Paradox</h3>
  <button id="rollParadox" style="padding: 10px 20px; font-size: 16px; cursor: pointer; background: #009900; color: white; border: none; border-radius: 4px; margin-bottom: 15px;">
    Paradox
  </button>
  <div id="paradoxResult" style="font-size: 18px; font-weight: bold; min-height: 25px; color: #000;"></div>
</div>

<!-- Section 2: Gary Larson (4 pairs) -->
<div style="margin: 20px 0; padding: 20px; border: 2px solid #333; border-radius: 8px; background: #f5f5f5;">
  <h3 style="margin-top: 0; color: #333;">Section 2: Gary Larson (4 Pairs)</h3>
  <button id="rollGaryLarson" style="padding: 10px 20px; font-size: 16px; cursor: pointer; background: #0066cc; color: white; border: none; border-radius: 4px; margin-bottom: 15px;">
    Gary Larson
  </button>
  <div id="garyLarsonResult" style="font-size: 18px; font-weight: bold; color: #000;"></div>
</div>

<!-- Section 3: Random Rolls (1-20 pairs) -->
<div style="margin: 20px 0; padding: 20px; border: 2px solid #333; border-radius: 8px; background: #f5f5f5;">
  <h3 style="margin-top: 0; color: #333;">Section 3: Random Chaos (1-20 Pairs)</h3>
  <button id="rollRandom" style="padding: 10px 20px; font-size: 16px; cursor: pointer; background: #cc6600; color: white; border: none; border-radius: 4px; margin-bottom: 15px;">
    Roll Random (1-20)
  </button>
  <div id="randomResult" style="font-size: 18px; font-weight: bold; color: #000;"></div>
</div>

<script>
(function() {
  const verbs = [
    "Shake", "Breath", "Abide", "Stabilize", "Expand",
    "Coordinate", "Engage", "Inhabit", "Withdraw", "Mu (Null)"
  ];

  const contexts = [
    "Self", "Engine", "Liquid", "Firmware", "Mu (Null)", "Context",
    "Other", "Network", "Transact", "Egregore", "Xenobiotic", "Terminate"
  ];

  // Helper function to generate a single paradox
  function generateParadox() {
    const verbRoll = Math.floor(Math.random() * 10) + 1;
    const contextRoll = Math.floor(Math.random() * 12) + 1;
    const verb = verbs[verbRoll - 1];
    const context = contexts[contextRoll - 1];
    return {
      verbRoll: verbRoll,
      verb: verb,
      contextRoll: contextRoll,
      context: context
    };
  }

  // Section 1: Single Paradox
  document.getElementById('rollParadox').addEventListener('click', function() {
    const paradox = generateParadox();
    document.getElementById('paradoxResult').innerHTML =
      `<div>Verb: ${paradox.verbRoll} - ${paradox.verb}</div>
       <div>Context: ${paradox.contextRoll} - ${paradox.context}</div>`;
  });

  // Section 2: Gary Larson (4 pairs)
  document.getElementById('rollGaryLarson').addEventListener('click', function() {
    let result = '';
    for (let i = 1; i <= 4; i++) {
      const paradox = generateParadox();
      result += `<div style="margin-bottom: 10px; padding: 10px; background: #fff; border-radius: 4px;">
                   <strong>Pair ${i}:</strong><br>
                   Verb: ${paradox.verbRoll} - ${paradox.verb}<br>
                   Context: ${paradox.contextRoll} - ${paradox.context}
                 </div>`;
    }
    document.getElementById('garyLarsonResult').innerHTML = result;
  });

  // Section 3: Random (1-20 pairs)
  document.getElementById('rollRandom').addEventListener('click', function() {
    const numPairs = Math.floor(Math.random() * 20) + 1;
    let result = `<div style="margin-bottom: 10px; color: #666;">Rolling ${numPairs} pairs...</div>`;

    for (let i = 1; i <= numPairs; i++) {
      const paradox = generateParadox();
      result += `<div style="margin-bottom: 8px; padding: 8px; background: #fff; border-radius: 4px;">
                   <strong>Pair ${i}:</strong>
                   Verb: ${paradox.verbRoll} - ${paradox.verb} |
                   Context: ${paradox.contextRoll} - ${paradox.context}
                 </div>`;
    }
    document.getElementById('randomResult').innerHTML = result;
  });
})();
</script>
{:/nomarkdown}                                                                                                                                                                                                                                                
             