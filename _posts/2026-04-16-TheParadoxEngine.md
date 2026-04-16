---
layout: post
draft: true
title: "The Paradox Engine"
description: 'A randomization tool I use to mix up my somatic practices.'
category: self
tags: [ proactive-prep, somatic work, parasympathetic, hrv, chaos-engineering ]
---

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

{::options parse_block_html="true" /}

<div style="margin: 20px 0; padding: 20px; border: 2px solid #333; border-radius: 8px; background: #f5f5f5;">
  <div style="display: flex; gap: 15px; margin-bottom: 15px;">
    <button id="rollVerb" style="padding: 10px 20px; font-size: 16px; cursor: pointer; background: #0066cc; color: white; border: none; border-radius: 4px;">
      Roll Verb (d10)
    </button>
    <button id="rollContext" style="padding: 10px 20px; font-size: 16px; cursor: pointer; background: #cc6600; color: white; border: none; border-radius: 4px;">
      Roll Context (d12)
    </button>
  </div>
  <div id="verbResult" style="margin-top: 10px; font-size: 18px; font-weight: bold; min-height: 25px;"></div>
  <div id="contextResult" style="margin-top: 10px; font-size: 18px; font-weight: bold; min-height: 25px;"></div>
</div>

<script>
const verbs = [
  "Shake", "Breath", "Abide", "Stabilize", "Expand",
  "Coordinate", "Engage", "Inhabit", "Withdraw", "Mu (Null)"
];

const contexts = [
  "Self", "Engine", "Liquid", "Firmware", "Mu (Null)", "Context",
  "Other", "Network", "Transact", "Egregore", "Xenobiotic", "Terminate"
];

document.getElementById('rollVerb').addEventListener('click', function() {
  const roll = Math.floor(Math.random() * 10) + 1;
  const verb = verbs[roll - 1];
  document.getElementById('verbResult').textContent = `Verb: ${roll} - ${verb}`;
});

document.getElementById('rollContext').addEventListener('click', function() {
  const roll = Math.floor(Math.random() * 12) + 1;
  const context = contexts[roll - 1];
  document.getElementById('contextResult').textContent = `Context: ${roll} - ${context}`;
});
</script>                                                                                                                                                                                                                                                
             