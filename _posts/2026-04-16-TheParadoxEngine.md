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

Click the button to generate a random verb-context pair:

  <div style="margin: 20px 0; padding: 20px; border: 2px solid #333; border-radius: 8px; background: #f5f5f5;">                                                                                                                                            
    <button id="rollButton" style="padding: 10px 20px; font-size: 16px; cursor: pointer; background: #0066cc; color: white; border: none; border-radius: 4px;">                                                                                            
      Roll the Dice                                                                                                                                                                                                                                        
    </button>                                                                                                                                                                                                                                              
    <div id="result" style="margin-top: 15px; font-size: 18px; font-weight: bold; min-height: 30px;"></div>                                                                                                                                                
  </div>                                                                                                                                                                                                                                                   

  <script>                                                                                                                                                                                                                                                 
  const verbs = [                                                                                                                                                                                                                                          
    "Discover", "Transform", "Challenge", "Navigate", "Innovate",                                                                                                                                                                                          
    "Create", "Explore", "Defend", "Unite", "Rebel"                                                                                                                                                                                                        
  ];                                                                                                                                                                                                                                                       
                                                                                                                                                                                                                                                           
  const contexts = [                                                                                                                                                                                                                                       
    "the ancient prophecy", "a hidden truth", "technological advancement",                                                                                                                                                                                 
    "political intrigue", "natural disaster", "personal redemption",                                                                                                                                                                                       
    "cultural conflict", "scientific breakthrough", "moral dilemma",                                                                                                                                                                                       
    "supernatural force", "economic collapse", "forbidden knowledge"                                                                                                                                                                                       
  ];                                                                                                                                                                                                                                                       
                                                                                                                                                                                                                                                           
  document.getElementById('rollButton').addEventListener('click', function() {                                                                                                                                                                             
    const verbRoll = Math.floor(Math.random() * 10);                                                                                                                                                                                                       
    const contextRoll = Math.floor(Math.random() * 12);                                                                                                                                                                                                    
                                                                                                                                                                                                                                                           
    const verb = verbs[verbRoll];                                                                                                                                                                                                                          
    const context = contexts[contextRoll];                                                                                                                                                                                                                 
                                                                                                                                                                                                                                                           
    document.getElementById('result').textContent = `${verb} ${context}`;                                                                                                                                                                                  
  });                                                                                                                                                                                                                                                      
  </script>                                                                                                                                                                                                                                                
             