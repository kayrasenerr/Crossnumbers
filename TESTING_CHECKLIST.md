# Manual testing checklist

Unity 2022.3.62f3, portrait Game view or WebGL browser.

1. Delete the `crossnumbers.save.v1` PlayerPrefs key (fresh install), open `Main`, press `NEW GAME`, and verify Level 1 shows a 5×5 board with `12345` and `54321`.
2. Drag a bank number. Every same-length slot should tint orange; hovering a target aligns the piece horizontally/vertically and previews green or conflict red. Drop to place. Settled locked digits return to blue and player digits to white. Tap a class header to inspect all slots of that length; repeat taps on a crossing to alternate vertical/horizontal focus.
3. Attempt a placement that disagrees with a blue digit. It should be rejected and return the picked piece to its bank without changing the fixed clue.
4. Place a crossing player number with a mismatching digit, then place the newer number. The newer placement should succeed and the older contradictory piece should return to the bank.
5. Drag a placed number to another same-length slot. Press Escape during the drag; the original placement should return. Drop outside the board to return it to the bank. At a crossing, tap to focus the intended orientation before beginning a move.
6. Use `Restart`; player placements and hint-created blue cells should clear, while authored blue cells remain.
7. Use Hint. Exactly one empty active cell becomes blue, the hint count decreases, and the cell remains locked after other pieces move.
8. Solve Levels 1–6 using the canonical assignments in `Docs/UPDATED SourceSpecs/UPDATED_level_data.md`. Completion should show the completed board, Congratulations, animated confetti, and +5 pass keys exactly once per level.
9. Return to Main during an unfinished level, verify the blue Continue / Level x button restores the board. The white New Game button underneath starts that same level fresh. For an unseen level, only the white New Game / Level x button appears.
10. Open Monthly Pass. Verify 30 steps, 0/30 initially, Free/Premium locked and claimable states, individual claims, Claim All, persistent hints, and mock Premium unlock with retroactive reached rewards.
11. Complete Level 6. The win overlay `NEW GAME` button should read `LEVEL 7 • COMING SOON 🔒` and do nothing. The Main `NEW GAME` button should also be inert after all six levels.
12. Refresh the WebGL page and verify progression, current board, hint balance, pass claims, premium state, settings, and cosmetics remain in browser storage.

13. Open Settings and press Help and Privacy Policy. Each should open a short centered modal with a dim backdrop and a working close/action button.
14. Open Monthly Pass and press `UNLOCK PREMIUM`. Confirm the iOS-style mock purchase dialog appears first; Cancel must leave Premium locked, while Confirm unlocks it and refreshes the pass.
15. On a level with authored blue digits, verify those cells have no pointer response and their complete authored number is present in the bank, dimmed as placed and non-interactive. Drag a bank number only; a tap without movement must not select it.
16. While a digit class or slot is highlighted, press a different-length active space. It should switch focus immediately. Press a blocked cell or background to clear focus. Drop a piece over a different-length slot or outside the board; it should return to the bank. Escape during an active drag restores its original placement.

Compilation and a focused Unity pointer-event drag-to-win smoke check passed during the UI repair. Screenshots are in `Validation/UI`. Full manual acceptance testing and WebGL browser testing remain for the user. Also resize the Game view between portrait and landscape: all controls should stay inside a centered portrait frame. Scroll the larger banks to reach every number, including all updated Level 6 pieces.


Updated visual/tutorial acceptance (2026-09-22):

17. Fresh Level 1: everything except 12345 and H1 is dimmed. The hand immediately repeats a 1-second travel / 2-second pause loop, with no initial wait for a press. A click alone must not place a piece.
18. Drag 12345 into H1; the guide switches to 54321 and V1. Place the second piece to remove the tutorial and win. Continue after the first placement should resume at the second tutorial step.
19. Hover a conflicting number: only individually conflicting cells should be pink; all compatible/empty positions must be green, on both the board and dragged preview.
20. Updated levels are 5x5, 6x6, 6x6, 8x8, 8x8, 9x9 with 2, 6, 6, 10, 10, 15 authored pieces. Old saves from changed layouts must start that level fresh without losing pass/hint/account progress.
21. Check Poppins, soft bank shadows, focus border/glow, flat grid fills, double board frame, settings cards/toggles, main puzzle decoration, radial win background, outlined gold heading and finite shaded ribbon confetti.
22. Monthly Pass: inspect all 30 milestones, scroll to the end, confirm/cancel mock Premium, claim individual/reached rewards and Claim all, revisit the page and verify persistent states. Premium styling must not change gameplay cell colors.

23. No guiding/status text should appear above the board, including after invalid drops. Both tutorial stages start the hand travel loop automatically.

### Clear all progress
- With level and Monthly Pass progress, open Settings > Clear all progress. Confirm the warning wording and red Clear label.
- Cancel (or close) must preserve all progress.
- Clear must return to Main with New Game for Level 1 (tutorial starts when opened), restore five starting hints, and remove Monthly Pass keys, claims, mock Premium ownership, and earned cosmetics.
- Return to Main and relaunch: Level 1 and the fresh pass must persist; sound/vibration preferences must remain unchanged.
