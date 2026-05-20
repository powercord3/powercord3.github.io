<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>My Websitmmme</title>
</head>
<body>
   import React, { useState, useEffect, useMemo } from 'react';
import { ChevronDown, Check, RotateCcw, ExternalLink } from 'lucide-react';

/* ----------------------------------------------------------------
   VERIFIED RESOURCES (checked May 2026)
------------------------------------------------------------------ */
const KA      = { label: 'Khan Academy — official Digital SAT course', type: 'learn',    url: 'https://www.khanacademy.org/digital-sat' };
const QB      = { label: 'Official SAT Question Bank — filter to this skill', type: 'practice', url: 'https://satsuitequestionbank.collegeboard.org/digital/search' };
const SQBORG  = { label: 'SATQuestionBank.org — timed practice with Desmos', type: 'practice', url: 'https://satquestionbank.org/' };
const RWSTEMS = { label: 'College Board — Reading & Writing question types', type: 'reference', url: 'https://satsuite.collegeboard.org/practice/student-question-bank/reading-writing' };
const BB      = { label: 'Bluebook — official full-length practice tests', type: 'test', url: 'https://bluebook.collegeboard.org/students/download-bluebook' };
const PAPER   = { label: 'Official paper practice tests (PDF)', type: 'test', url: 'https://satsuite.collegeboard.org/practice/practice-tests/paper' };
const DESMOS  = { label: 'Desmos — graphing calculator', type: 'tool', url: 'https://www.desmos.com/calculator' };

const resTag = {
  learn:     { t: 'Learn',    c: '#3d5236' },
  practice:  { t: 'Practice', c: '#7a1c2b' },
  test:      { t: 'Test',     c: '#33396b' },
  tool:      { t: 'Tool',     c: '#7a4a1c' },
  reference: { t: 'Guide',    c: '#2a5252' },
};

/* ----------------------------------------------------------------
   CONTENT LIBRARY  (review = custom facts + self-check questions)
------------------------------------------------------------------ */
const DOMAINS = {
  algebra: { label: 'Algebra', side: 'math', pillFocus: 'Algebra', lessons: [
    { id: 'alg-linear',   focus: 'Algebra', title: 'Linear Equations & Inequalities', tasks: ['Solve linear equations and inequalities in one and two variables', 'Translate word problems into clean linear models', 'Work 20 Heart of Algebra problems and log every miss'] },
    { id: 'alg-systems',  focus: 'Algebra', title: 'Systems of Equations',             tasks: ['Solve systems by substitution and elimination', 'Recognize zero, one, or infinitely many solutions', 'Practice 15 system problems'],
      review: { facts: ['One solution: lines cross once. No solution: parallel (same slope, different intercept). Infinite: identical lines.', 'Elimination: scale equations so one variable cancels when added.'],
        questions: [{ q: 'How many solutions: y = 2x + 1 and y = 2x − 4?', a: 'None — same slope, different intercept, so the lines are parallel.' }] } },
    { id: 'alg-functions',focus: 'Algebra', title: 'Linear Functions & Graphs',         tasks: ['Move fluently between slope-intercept and standard form', 'Interpret slope and intercepts in context', 'Match 12 equations to their graphs'] },
    { id: 'alg-absval',   focus: 'Algebra', title: 'Absolute Value & Inequalities',     tasks: ['Solve absolute value equations and inequalities', 'Graph solution sets on a number line', 'Practice 10 problems'] },
  ]},
  advmath: { label: 'Advanced Math', side: 'math', pillFocus: 'Advanced', lessons: [
    { id: 'adv-quad',   focus: 'Advanced', title: 'Quadratics & Parabolas',    tasks: ['Factor, complete the square, and apply the quadratic formula', 'Find vertex, axis of symmetry, and intercepts', 'Work 15 quadratic problems'],
      review: { facts: ['Quadratic formula: x = (−b ± √(b² − 4ac)) / 2a.', 'Discriminant b² − 4ac: positive = 2 real roots, zero = 1, negative = 0.', 'Vertex of y = ax² + bx + c is at x = −b / 2a.'],
        questions: [{ q: 'How many real solutions does x² + 4x + 5 = 0 have?', a: 'Zero — discriminant = 16 − 20 = −4 < 0.' }, { q: 'Vertex x-coordinate of y = 2x² − 8x + 1?', a: 'x = 2 — using −b/2a = 8/4.' }] } },
    { id: 'adv-poly',   focus: 'Advanced', title: 'Polynomials & Factoring',   tasks: ['Add, subtract, and multiply polynomials', 'Factor by grouping and recognize special forms', 'Practice 12 problems'] },
    { id: 'adv-exprad', focus: 'Advanced', title: 'Exponents & Radicals',      tasks: ['Apply exponent rules and rational exponents', 'Solve radical and exponential equations', 'Practice 12 problems'] },
    { id: 'adv-func',   focus: 'Advanced', title: 'Functions & Notation',      tasks: ['Evaluate and compose functions confidently', 'Read function behavior straight off a graph', 'Work 12 function problems'] },
  ]},
  psda: { label: 'Problem-Solving & Data', side: 'math', pillFocus: 'Data', lessons: [
    { id: 'ps-ratios',  focus: 'Data', title: 'Ratios, Rates & Proportions',   tasks: ['Set up and solve proportions quickly', 'Handle unit rates and unit conversions', 'Practice 15 problems'] },
    { id: 'ps-percent', focus: 'Data', title: 'Percentages & Percent Change',   tasks: ['Percent increase, decrease, and reversal', 'Distinguish simple from compound change', 'Work 12 percent problems'],
      review: { facts: ['Percent change = (new − old) / old × 100.', 'A 20% increase then 20% decrease does NOT return to the start (0.8 × 1.2 = 0.96).'],
        questions: [{ q: 'A $50 item rises 10%, then falls 10%. Final price?', a: '$49.50 — 50 × 1.1 × 0.9.' }] } },
    { id: 'ps-data',    focus: 'Data', title: 'Tables, Graphs & Charts',        tasks: ['Read scatterplots, bar graphs, and two-way tables', 'Interpret lines of best fit and trends', 'Answer 15 questions across chart types'] },
    { id: 'ps-stats',   focus: 'Data', title: 'Statistics — Center & Spread',   tasks: ['Compute mean, median, mode, and range', 'Predict how outliers shift mean vs. median', 'Practice 12 statistics problems'],
      review: { facts: ['Mean is pulled toward outliers; median resists them.', 'Range = max − min. Standard deviation measures spread around the mean.'],
        questions: [{ q: 'Which moves more if you add one huge value to a data set — mean or median?', a: 'The mean — it is sensitive to outliers; the median barely shifts.' }] } },
  ]},
  geomtrig: { label: 'Geometry & Trig', side: 'math', pillFocus: 'Geometry', lessons: [
    { id: 'gt-lines',     focus: 'Geometry', title: 'Lines, Angles & Triangles',           tasks: ['Angle relationships: vertical, supplementary, parallel-line angles', 'Triangle angle sum (180°) and the exterior angle theorem', 'Sketch and solve 12 angle-chasing problems'],
      review: { facts: ['Angles in a triangle sum to 180°.', 'Exterior angle = sum of the two remote interior angles.', 'Parallel lines cut by a transversal: corresponding and alternate angles are equal.'],
        questions: [{ q: 'A triangle has angles 40° and 75°. The third angle?', a: '65° — 180 − 40 − 75.' }] } },
    { id: 'gt-triangles', focus: 'Geometry', title: 'Triangles in Depth',                  tasks: ['Pythagorean theorem and triples (3-4-5, 5-12-13, 8-15-17)', 'Special right triangles: 30-60-90 and 45-45-90 ratios', 'Similar vs. congruent triangles and proportional sides', 'Work 15 mixed triangle problems'],
      review: { facts: ['Pythagorean theorem: a² + b² = c² (c is the hypotenuse).', 'Common triples: 3-4-5, 5-12-13, 8-15-17 and their multiples.', '30-60-90 ratio = 1 : √3 : 2 (short : long : hypotenuse).', '45-45-90 ratio = 1 : 1 : √2 (leg : leg : hypotenuse).'],
        questions: [{ q: 'Right triangle with legs 9 and 12 — hypotenuse?', a: '15 — a 3-4-5 triple scaled by 3.' }, { q: '30-60-90 triangle, hypotenuse 14 — side opposite 30°?', a: '7 — the short leg is half the hypotenuse.' }, { q: '45-45-90 triangle with legs of 6 — hypotenuse?', a: '6√2.' }] } },
    { id: 'gt-polygons',  focus: 'Geometry', title: 'Polygons, Quadrilaterals & Area',     tasks: ['Interior angle sum: (n − 2) × 180°', 'Properties of parallelograms, rectangles, rhombuses, trapezoids', 'Memorize area formulas for all common shapes', 'Practice 12 area-and-angle problems'],
      review: { facts: ['Interior angle sum of an n-gon: (n − 2) × 180°.', 'Each angle of a regular n-gon: (n − 2) × 180° ÷ n.', 'Triangle area = ½ bh. Trapezoid = ½ (b₁ + b₂) h. Parallelogram = bh.'],
        questions: [{ q: 'Sum of interior angles of a hexagon?', a: '720° — (6 − 2) × 180°.' }, { q: 'Each interior angle of a regular pentagon?', a: '108° — 540° ÷ 5.' }] } },
    { id: 'gt-trig1',     focus: 'Trig',     title: 'Right Triangle Trigonometry',         tasks: ['Drill SOHCAHTOA until it is automatic', 'Identify opposite, adjacent, and hypotenuse from any angle', 'Solve 15 right-triangle trig problems'],
      review: { facts: ['SOH: sin θ = opposite / hypotenuse.', 'CAH: cos θ = adjacent / hypotenuse.', 'TOA: tan θ = opposite / adjacent.'],
        questions: [{ q: 'Opposite side 3, hypotenuse 5 — what is sin θ?', a: '3/5 = 0.6.' }, { q: 'If tan θ = 4/3, give possible opposite and adjacent sides.', a: 'Opposite 4, adjacent 3 (a 3-4-5 triangle).' }] } },
    { id: 'gt-trig2',     focus: 'Trig',     title: 'Special Angles & Cofunctions',        tasks: ['Memorize exact sin, cos, tan at 30°, 45°, 60°', 'Learn the cofunction identity: sin θ = cos(90° − θ)', 'Practice 12 exact-value and cofunction problems'],
      review: { facts: ['sin: 30°→½, 45°→√2/2, 60°→√3/2.', 'cos: 30°→√3/2, 45°→√2/2, 60°→½.', 'tan: 30°→√3/3, 45°→1, 60°→√3.', 'Cofunction: sin θ = cos(90° − θ).'],
        questions: [{ q: 'If sin x° = cos 35°, find x (0 < x < 90).', a: '55 — since sin x = cos(90 − x), x = 55.' }, { q: 'Exact value of cos 60°?', a: '½.' }] } },
    { id: 'gt-unit',      focus: 'Trig',     title: 'Radians & The Unit Circle',           tasks: ['Convert between degrees and radians fluently', 'Map sin and cos at every key unit-circle angle', 'Practice 10 problems mixing radians and trig'],
      review: { facts: ['180° = π radians, so degrees × π/180 = radians.', 'On the unit circle a point is (cos θ, sin θ).'],
        questions: [{ q: 'Convert 90° to radians.', a: 'π/2.' }, { q: 'Convert 2π/3 radians to degrees.', a: '120°.' }] } },
    { id: 'gt-circ1',     focus: 'Geometry', title: 'Circles — Lengths & Areas',          tasks: ['Area (πr²) and circumference (2πr)', 'Arc length and sector area from a central angle', 'Work 12 arc-and-sector problems'],
      review: { facts: ['Circumference = 2πr; Area = πr².', 'Arc length = (angle / 360°) × 2πr.', 'Sector area = (angle / 360°) × πr².'],
        questions: [{ q: 'Radius 6, central angle 60° — arc length?', a: '2π — (60/360)(12π) = 2π.' }, { q: 'Radius 4, 90° sector — area?', a: '4π — (90/360)(16π).' }] } },
    { id: 'gt-circ2',     focus: 'Geometry', title: 'Circles — Equations in the Plane',   tasks: ['Standard form (x − h)² + (y − k)² = r²', 'Complete the square to find center and radius', 'Solve 12 circle-equation problems'],
      review: { facts: ['Standard form: (x − h)² + (y − k)² = r², center (h, k), radius r.', 'Complete the square on the x-terms and y-terms to convert from general form.'],
        questions: [{ q: 'Center and radius of (x − 3)² + (y + 2)² = 25?', a: 'Center (3, −2), radius 5.' }, { q: 'x² + y² − 6x = 16 — center and radius?', a: '(x − 3)² + y² = 25 → center (3, 0), radius 5.' }] } },
    { id: 'gt-3d',        focus: 'Geometry', title: '3-D Geometry & Volume',               tasks: ['Volume of prisms, cylinders, pyramids, cones, spheres', 'Surface area of common solids', 'Solve 10 problems with 3-D solids'],
      review: { facts: ['Cylinder volume = πr²h. Cone = ⅓πr²h. Sphere = 4/3 πr³.', 'A cone is exactly one-third of the cylinder with the same base and height.'],
        questions: [{ q: 'Cylinder radius 3, height 10 — volume?', a: '90π — π(3²)(10).' }] } },
    { id: 'gt-coord',     focus: 'Geometry', title: 'Coordinate Geometry',                 tasks: ['Distance and midpoint formulas', 'Slope, parallel and perpendicular lines', 'Equations of lines in multiple forms', 'Work 12 coordinate-geometry problems'],
      review: { facts: ['Distance = √[(x₂ − x₁)² + (y₂ − y₁)²]. Midpoint = ((x₁+x₂)/2, (y₁+y₂)/2).', 'Parallel lines share a slope; perpendicular slopes multiply to −1.'],
        questions: [{ q: 'Slope perpendicular to a line with slope 2/3?', a: '−3/2.' }, { q: 'Distance between (1, 2) and (4, 6)?', a: '5 — √(3² + 4²).' }] } },
  ]},
  infoideas:   { label: 'Information & Ideas', side: 'reading', pillFocus: 'Reading', lessons: [
    { id: 'ri-main',     focus: 'Reading', title: 'Grabbing Main Ideas Fast',      tasks: ['Run the 30-second drill: read first and last sentence of each paragraph', 'State each passage thesis in a single sentence', 'Practice on 3 short passages, timing yourself'],
      review: { facts: ['The main idea is the claim the whole passage supports — not one striking detail.', 'Predict an answer in your own words before reading the choices.'],
        questions: [{ q: 'A choice that is true but covers only one paragraph of a long passage is usually wrong because it is…?', a: 'Too narrow — main-idea answers must cover the whole passage.' }] } },
    { id: 'ri-details',  focus: 'Reading', title: 'Finding Details Efficiently',   tasks: ['For line references, read 2 sentences above and 2 below — rarely just the line', 'Locate the supporting detail without re-reading the whole passage', 'Drill 10 detail questions under time'] },
    { id: 'ri-evidence', focus: 'Reading', title: 'Command of Evidence',           tasks: ['Match each claim to the line that best supports it', 'Interpret quantitative evidence in graphs paired with text', 'Practice 10 evidence questions'] },
    { id: 'ri-infer',    focus: 'Reading', title: 'Inferences',                    tasks: ['Separate stated fact from logical inference', 'Choose the most supported conclusion, not the most interesting', 'Work 10 inference questions'] },
  ]},
  craft: { label: 'Craft & Structure', side: 'reading', pillFocus: 'Reading', lessons: [
    { id: 'cs-vocab',     focus: 'Reading', title: 'Words in Context',          tasks: ['Use surrounding sentences to pin down meaning', 'Watch for secondary meanings of familiar words', 'Practice 15 vocabulary-in-context questions'] },
    { id: 'cs-structure', focus: 'Reading', title: 'Text Structure & Purpose',  tasks: ['Identify how a paragraph functions in the whole', 'Spot purpose: argue, illustrate, qualify, or contrast', 'Work 10 structure questions'] },
    { id: 'cs-crosstext', focus: 'Reading', title: 'Cross-Text Connections',    tasks: ['Compare the stance of two paired passages', 'Predict how one author would respond to the other', 'Practice 8 paired-passage questions'] },
  ]},
  expression: { label: 'Expression of Ideas', side: 'reading', pillFocus: 'Writing', lessons: [
    { id: 'ei-synth', focus: 'Writing', title: 'Rhetorical Synthesis',       tasks: ['Use bulleted notes to hit one specific writing goal', 'Pick the choice that satisfies the stated objective only', 'Practice 12 synthesis questions'] },
    { id: 'ei-trans', focus: 'Writing', title: 'Transitions & Logical Flow', tasks: ['Sort transitions: contrast, cause, addition, sequence', 'Test the logical link between each pair of sentences', 'Work 12 transition questions'],
      review: { facts: ['Decide the logical relationship first, then choose the transition.', 'Contrast: however, but. Cause/effect: therefore, as a result. Addition: moreover. Sequence: then, finally.'],
        questions: [{ q: 'Sentence 1 states a problem; sentence 2 gives the fix. Best transition type?', a: 'Cause/effect or resolution — e.g. "therefore" or "as a result."' }] } },
  ]},
  conventions: { label: 'Standard English Conventions', side: 'reading', pillFocus: 'Writing', lessons: [
    { id: 'sc-punct', focus: 'Writing', title: 'Semicolons, Colons & Dashes', tasks: ['Semicolons join two independent clauses — learn when a comma is wrong', 'Colons and em dashes after a complete clause', 'Practice 15 punctuation questions'],
      review: { facts: ['Semicolon joins two complete sentences: "I studied hard; I passed."', 'A colon follows a complete clause to introduce a list or explanation.', 'A single dash adds emphasis or introduces a list after a complete clause.', 'A comma alone cannot join two complete sentences (comma splice).'],
        questions: [{ q: '"The lab ran late ___ we finished anyway." Both halves are complete. Which mark?', a: 'A semicolon (;) — it joins two independent clauses.' }, { q: 'Correct? "She packed three things: a map, snacks, and water."', a: 'Yes — a colon after a complete clause properly introduces a list.' }] } },
    { id: 'sc-bound', focus: 'Writing', title: 'Sentence Boundaries',         tasks: ['Spot comma splices, run-ons, and fragments instantly', 'Fix boundaries with the four correct punctuation moves', 'Work 12 boundary problems'],
      review: { facts: ['Comma splice = two complete sentences joined by only a comma.', 'Fix it with a period, a semicolon, or a comma + FANBOYS (for, and, nor, but, or, yet, so).', 'A fragment is missing a subject or a verb.'],
        questions: [{ q: '"The rain stopped, we went outside." Fix the comma splice.', a: 'Use a period or semicolon, or add a conjunction: "…stopped, so we went outside."' }, { q: 'Is "Because the test was hard." a complete sentence?', a: 'No — it is a fragment (a dependent clause with no main clause).' }] } },
    { id: 'sc-agree', focus: 'Writing', title: 'Agreement & Modifiers',       tasks: ['Subject-verb agreement with intervening phrases', 'Pronoun agreement and clear reference', 'Dangling and misplaced modifiers', 'Practice 15 problems'],
      review: { facts: ['The verb agrees with the true subject, not a noun in a phrase between them.', '"The box of nails is heavy" — subject is "box," not "nails."', 'A modifier must sit beside the word it describes.'],
        questions: [{ q: 'Choose: "The collection of rare coins (was / were) stolen."', a: '"was" — the subject "collection" is singular.' }, { q: 'Fix: "Running late, the bus was missed by Tom."', a: 'Dangling modifier — "Running late, Tom missed the bus."' }] } },
  ]},
};

const MATH_KEYS = ['algebra', 'advmath', 'psda', 'geomtrig'];
const READ_KEYS = ['infoideas', 'craft', 'expression', 'conventions'];
const ALL_KEYS  = [...MATH_KEYS, ...READ_KEYS];

const DEFAULT_MATH_WEIGHTS = { algebra: 15, advmath: 15, psda: 15, geomtrig: 55 };
const DEFAULT_READ_WEIGHTS  = { infoideas: 30, craft: 20, expression: 20, conventions: 30 };

const focusStyles = {
  Audit:    { bg: '#f0ece2', fg: '#4a443e' },
  Algebra:  { bg: '#e8eee4', fg: '#3d5236' },
  Advanced: { bg: '#e0e2f0', fg: '#33396b' },
  Data:     { bg: '#e6ecf0', fg: '#385164' },
  Geometry: { bg: '#f3e6e8', fg: '#7a1c2b' },
  Trig:     { bg: '#f0e4d9', fg: '#7a4a1c' },
  Practice: { bg: '#ece4f0', fg: '#4a2b6b' },
  Review:   { bg: '#eee9df', fg: '#5a503e' },
  Test:     { bg: '#3a1a1f', fg: '#faf0e6' },
  Prep:     { bg: '#1a1614', fg: '#faf0e6' },
  Writing:  { bg: '#dce7e7', fg: '#2a5252' },
  Reading:  { bg: '#e5e8d2', fg: '#4a5a2a' },
};

// Default resource set per side (used unless a lesson defines its own).
const lessonResources = (lesson, side) =>
  lesson.resources || (side === 'math' ? [KA, QB, SQBORG] : [KA, QB, RWSTEMS]);

/* ----------------------------------------------------------------
   CONSTRAINED SLIDER GROUP
------------------------------------------------------------------ */
function adjustGroup(keys, currentWeights, changedKey, rawNewVal) {
  const newVal = Math.max(0, Math.min(100, Math.round(rawNewVal)));
  const others = keys.filter((k) => k !== changedKey);
  const remainingBudget = 100 - newVal;
  if (newVal >= 100) {
    const r = { [changedKey]: 100 };
    others.forEach((k) => { r[k] = 0; });
    return r;
  }
  const othersSum = others.reduce((a, k) => a + currentWeights[k], 0);
  let shares;
  if (othersSum <= 0) {
    const base = Math.floor(remainingBudget / others.length);
    const extra = remainingBudget - base * others.length;
    shares = others.map((_, i) => base + (i < extra ? 1 : 0));
  } else {
    const exact = others.map((k) => (currentWeights[k] / othersSum) * remainingBudget);
    const floored = exact.map((v) => Math.floor(v));
    let leftover = remainingBudget - floored.reduce((a, b) => a + b, 0);
    exact.map((v, i) => ({ i, frac: v - Math.floor(v) })).sort((a, b) => b.frac - a.frac)
      .forEach(({ i }) => { if (leftover > 0) { floored[i]++; leftover--; } });
    shares = floored;
  }
  const result = { [changedKey]: newVal };
  others.forEach((k, i) => { result[k] = shares[i]; });
  return result;
}

function normalizeGroup(keys, rawWeights, fallback) {
  const sum = keys.reduce((a, k) => a + Math.max(0, rawWeights[k] || 0), 0);
  if (sum <= 0) return { ...fallback };
  const exact = keys.map((k) => ((rawWeights[k] || 0) / sum) * 100);
  const floored = exact.map(Math.floor);
  let leftover = 100 - floored.reduce((a, b) => a + b, 0);
  exact.map((v, i) => ({ i, frac: v - Math.floor(v) })).sort((a, b) => b.frac - a.frac)
    .forEach(({ i }) => { if (leftover > 0) { floored[i]++; leftover--; } });
  const result = {};
  keys.forEach((k, i) => { result[k] = floored[i]; });
  return result;
}

/* ----------------------------------------------------------------
   PLAN GENERATOR
------------------------------------------------------------------ */
const structDiagnostic = () => ({ id: 'struct-diag', focus: 'Audit', title: 'Diagnostic & Setup', tasks: ['Take a full untimed practice section to set a baseline', 'Mark every miss by question type', 'Set up a topic-organized error notebook'], resources: [BB] });
const structTest   = (n) => ({ id: `struct-test${n}`, focus: 'Test', title: n > 1 ? `Full Timed Section — Test ${n}` : 'Full Timed Section', tasks: ['Take a full section under exact timing, phone in another room', 'Score it and write down every missed question', 'Tag each miss: concept, careless, or pacing'], resources: [BB, PAPER] });
const structReview = (n) => ({ id: `struct-review${n}`, focus: 'Review', title: 'Review & Targeted Drill', tasks: ['Redo every missed problem without looking at solutions', 'Drill 10 extra problems on your single weakest area', 'Update your formula and grammar-rules sheet'], resources: [QB, KA] });
const structPrep   = ()  => ({ id: 'struct-prep', focus: 'Prep', title: 'Final Prep', tasks: ['Re-read your formula sheet and English conventions notes', 'Re-do a small handful of your favorite tricky problems', 'Pack everything you need for test day tonight', 'Sleep early — you have done the work'], resources: [PAPER, DESMOS] });
const drillDay = (key, n) => ({ id: `${key}-drill${n}`, focus: 'Practice', title: `${DOMAINS[key].label} — Timed Drill`, tasks: [`Timed mixed set focused on ${DOMAINS[key].label}`, 'Review every miss in writing before moving on', 'Note any sub-topic that still feels shaky'], resources: [QB, SQBORG] });

function allocateDomains(total, keys, weights) {
  if (total <= 0 || keys.length === 0) return [];
  const ws = keys.map((k) => weights[k]);
  const wsum = ws.reduce((a, b) => a + b, 0);
  if (wsum <= 0) return [];
  const exact = keys.map((k, i) => (total * ws[i]) / wsum);
  const alloc = keys.map((k, i) => ({ key: k, days: Math.floor(exact[i]) }));
  let rem = total - alloc.reduce((a, b) => a + b.days, 0);
  keys.map((k, i) => ({ i, frac: exact[i] - Math.floor(exact[i]) })).sort((a, b) => b.frac - a.frac)
      .forEach(({ i }) => { if (rem > 0) { alloc[i].days++; rem--; } });
  return alloc;
}

function pullLessons(alloc) {
  const out = [];
  alloc.forEach(({ key, days }) => {
    if (days <= 0) return;
    const pool = DOMAINS[key].lessons;
    const side = DOMAINS[key].side;
    for (let i = 0; i < days; i++) {
      if (i < pool.length) out.push({ ...pool[i], domain: key, resources: lessonResources(pool[i], side) });
      else out.push(drillDay(key, i - pool.length + 1));
    }
  });
  return out;
}

function interleave(a, b) {
  if (!a.length) return b.slice();
  if (!b.length) return a.slice();
  const out = [];
  let ai = 0, bi = 0;
  while (ai < a.length || bi < b.length) {
    const aProg = ai / a.length, bProg = bi / b.length;
    if (ai < a.length && (bi >= b.length || aProg <= bProg)) out.push(a[ai++]);
    else out.push(b[bi++]);
  }
  return out;
}

const WEEK_WORDS = ['One','Two','Three','Four','Five','Six','Seven','Eight','Nine','Ten'];

function buildPlan({ totalDays, mathPct, mathWeights, readWeights }) {
  const mathDomains = MATH_KEYS.filter((k) => mathWeights[k] > 0);
  const readDomains = READ_KEYS.filter((k) => readWeights[k]  > 0);
  const hasMath = mathDomains.length > 0;
  const hasRead = readDomains.length > 0;
  if (!hasMath && !hasRead) return { weeks: [], empty: true, counts: null };

  const diagnostic = totalDays >= 6 ? 1 : 0;
  const prep       = totalDays >= 3 ? 1 : 0;
  let testPairs = 0, singleTest = 0;
  if      (totalDays >= 40) testPairs = 3;
  else if (totalDays >= 18) testPairs = 2;
  else if (totalDays >= 11) testPairs = 1;
  else if (totalDays >= 7)  singleTest = 1;
  const testDays = testPairs * 2 + singleTest;

  const contentDays = Math.max(0, totalDays - diagnostic - prep - testDays);
  const m = mathPct / 100;
  let mathContent, readContent;
  if (hasMath && hasRead) {
    mathContent = Math.round(contentDays * m);
    readContent = contentDays - mathContent;
    if (mathContent === 0 && contentDays > 0 && m > 0)  { mathContent = 1; readContent--; }
    if (readContent  === 0 && contentDays > 0 && m < 1) { readContent  = 1; mathContent--; }
  } else if (hasMath) { mathContent = contentDays; readContent = 0; }
  else                { readContent = contentDays; mathContent = 0; }

  const allWeights = { ...mathWeights, ...readWeights };
  const mathSeq = pullLessons(allocateDomains(mathContent, mathDomains, allWeights));
  const readSeq = pullLessons(allocateDomains(readContent,  readDomains,  allWeights));
  const contentSeq = interleave(mathSeq, readSeq);

  const days = [];
  if (diagnostic) days.push(structDiagnostic());
  days.push(...contentSeq);
  for (let i = 0; i < testPairs; i++) { days.push(structTest(i + 1)); days.push(structReview(i + 1)); }
  if (singleTest) days.push(structTest(1));
  if (prep) days.push(structPrep());
  days.forEach((d, idx) => { d.day = `Day ${idx + 1}`; });

  const weeks = [];
  for (let i = 0; i < days.length; i += 7) {
    const slice = days.slice(i, i + 7);
    const wn = Math.floor(i / 7);
    const focuses = [...new Set(slice.map((d) => d.focus))].slice(0, 3);
    weeks.push({ id: `w${wn + 1}`, weekNum: WEEK_WORDS[wn] || String(wn + 1), title: focuses.join('  ·  ') || 'Study', days: slice });
  }
  return { weeks, empty: false, counts: { mathContent, readContent, diagnostic, testDays, prep, total: days.length } };
}

/* ----------------------------------------------------------------
   COMPONENTS
------------------------------------------------------------------ */
function CheckBox({ state, onClick, size = 'md' }) {
  const dim = { sm: 'w-4 h-4', md: 'w-5 h-5', lg: 'w-6 h-6' }[size];
  return (
    <button onClick={(e) => { e.stopPropagation(); onClick(); }}
      className={`${dim} flex items-center justify-center rounded-[3px] border transition-all duration-150 shrink-0`}
      style={{ borderColor: state === 'empty' ? 'var(--line-strong)' : 'var(--accent)', background: state === 'full' ? 'var(--accent)' : 'transparent' }}
      aria-label="toggle">
      {state === 'full'    && <Check className="w-3.5 h-3.5" style={{ color: 'var(--bg)', strokeWidth: 3 }} />}
      {state === 'partial' && <div className="w-1.5 h-1.5 rounded-full" style={{ background: 'var(--accent)' }} />}
    </button>
  );
}

function SectionSlider({ value, label, focus, onChange }) {
  const f = focusStyles[focus] || focusStyles.Review;
  const off = value <= 0;
  const trackBg = off ? 'var(--line-strong)'
    : `linear-gradient(to right, ${f.fg} 0%, ${f.fg} ${value}%, var(--line-strong) ${value}%, var(--line-strong) 100%)`;
  return (
    <div>
      <div className="flex items-baseline justify-between mb-1.5 gap-2">
        <span className="text-[13px] font-medium leading-tight" style={{ color: off ? 'var(--muted)' : 'var(--ink-soft)', textDecoration: off ? 'line-through' : 'none' }}>{label}</span>
        <span className="font-mono text-[12px] tabular-nums shrink-0 w-10 text-right" style={{ color: off ? 'var(--muted)' : f.fg }}>{off ? 'off' : `${value}%`}</span>
      </div>
      <input type="range" min={0} max={100} step={1} value={value} onChange={(e) => onChange(Number(e.target.value))}
        className="sat-range sat-range-sec" style={{ background: trackBg, '--sl': f.fg }} />
    </div>
  );
}

function ResourceLink({ res }) {
  const tag = resTag[res.type] || resTag.learn;
  return (
    <a href={res.url} target="_blank" rel="noopener noreferrer"
      className="group flex items-center gap-2.5 py-1.5"
      onClick={(e) => e.stopPropagation()}>
      <span className="font-mono text-[9px] tracking-[0.08em] uppercase px-1.5 py-0.5 rounded-sm shrink-0"
        style={{ background: 'var(--surface)', color: tag.c, border: `1px solid ${tag.c}33` }}>{tag.t}</span>
      <span className="text-[13px] leading-snug flex-1" style={{ color: 'var(--ink-soft)' }}>{res.label}</span>
      <ExternalLink className="w-3 h-3 shrink-0 opacity-40 group-hover:opacity-80 transition-opacity" style={{ color: 'var(--accent)' }} />
    </a>
  );
}

function ReviewQuestion({ q, a }) {
  const [open, setOpen] = useState(false);
  return (
    <li>
      <button onClick={(e) => { e.stopPropagation(); setOpen((o) => !o); }} className="text-left w-full flex items-start gap-2 py-1.5">
        <span className="font-mono text-[10px] mt-[3px] shrink-0" style={{ color: 'var(--accent)' }}>Q</span>
        <span className="flex-1 text-[13px] leading-snug" style={{ color: 'var(--ink-soft)' }}>
          {q}
          <span className="font-mono text-[10px] ml-2" style={{ color: 'var(--muted)' }}>{open ? '' : '· tap to reveal'}</span>
        </span>
      </button>
      <div className="sat-collapse" style={{ gridTemplateRows: open ? '1fr' : '0fr' }}>
        <div className="sat-collapse-inner">
          <div className="flex items-start gap-2 pb-2 pl-[1.1rem]">
            <span className="font-mono text-[10px] mt-[3px] shrink-0" style={{ color: 'var(--muted)' }}>A</span>
            <span className="flex-1 text-[13px] leading-snug" style={{ color: 'var(--ink)' }}>{a}</span>
          </div>
        </div>
      </div>
    </li>
  );
}

function DayDetail({ day }) {
  const review = day.review;
  return (
    <div className="pl-[3.4rem] md:pl-[3.7rem] pr-6 pb-5 pt-2" style={{ background: 'var(--bg)' }}>
      {/* Tasks */}
      <ul className="space-y-2.5 mb-4">
        {day.tasks.map((task, ti) => (
          <li key={ti} className="flex items-start gap-3">
            <span className="mt-[7px] w-1 h-1 rounded-full shrink-0" style={{ background: 'var(--line-strong)' }} />
            <span className="text-[15px] leading-relaxed flex-1" style={{ color: 'var(--ink-soft)' }}>{task}</span>
          </li>
        ))}
      </ul>

      {/* Custom quick review */}
      {review && (
        <div className="mb-4 rounded-sm border p-3.5" style={{ background: 'var(--surface)', borderColor: 'var(--line)' }}>
          <div className="font-mono text-[9px] tracking-[0.18em] uppercase mb-2" style={{ color: 'var(--accent)' }}>Quick review</div>
          <ul className="space-y-1.5 mb-2">
            {review.facts.map((fact, fi) => (
              <li key={fi} className="flex items-start gap-2">
                <span className="mt-[7px] w-1 h-1 rounded-full shrink-0" style={{ background: 'var(--accent)' }} />
                <span className="text-[13px] leading-snug flex-1" style={{ color: 'var(--ink)' }}>{fact}</span>
              </li>
            ))}
          </ul>
          {review.questions?.length > 0 && (
            <ul className="border-t pt-1.5" style={{ borderColor: 'var(--line)' }}>
              {review.questions.map((qq, qi) => <ReviewQuestion key={qi} q={qq.q} a={qq.a} />)}
            </ul>
          )}
        </div>
      )}

      {/* Resources */}
      {day.resources?.length > 0 && (
        <div>
          <div className="font-mono text-[9px] tracking-[0.18em] uppercase mb-1" style={{ color: 'var(--muted)' }}>
            {day.resources.length === 1 ? 'Where to do this' : 'Recommended sources'}
          </div>
          <div className="divide-y" style={{ borderColor: 'var(--line)' }}>
            {day.resources.map((res, ri) => <ResourceLink key={ri} res={res} />)}
          </div>
          {day.resources.some((r) => r.url === QB.url) && (
            <p className="font-mono text-[10px] mt-1.5" style={{ color: 'var(--muted)' }}>
              In the Question Bank, filter by Test → Domain → Skill → Difficulty to target this exact topic.
            </p>
          )}
        </div>
      )}
    </div>
  );
}

/* ----------------------------------------------------------------
   MAIN
------------------------------------------------------------------ */
const STORAGE_KEY = 'sat-plan:progress-v3';
const taskId = (dayId, idx) => `${dayId}t${idx}`;

const DEFAULT_CONTROLS = { totalDays: 18, mathPct: 70, mathWeights: { ...DEFAULT_MATH_WEIGHTS }, readWeights: { ...DEFAULT_READ_WEIGHTS } };

export default function StudyPlan() {
  const [controls, setControls]   = useState(DEFAULT_CONTROLS);
  const [completed, setCompleted] = useState({});
  const [openWeeks, setOpenWeeks] = useState({ w1: true });
  const [openDays,  setOpenDays]  = useState({});
  const [loaded,    setLoaded]    = useState(false);

  useEffect(() => {
    (async () => {
      try {
        if (typeof window !== 'undefined' && window.storage) {
          const res = await window.storage.get(STORAGE_KEY);
          if (res?.value) {
            const d = JSON.parse(res.value);
            if (d.controls) {
              let mw, rw;
              if (d.controls.mathWeights && d.controls.readWeights) {
                mw = { ...DEFAULT_MATH_WEIGHTS, ...d.controls.mathWeights };
                rw = { ...DEFAULT_READ_WEIGHTS,  ...d.controls.readWeights };
              } else if (d.controls.weights) {
                mw = normalizeGroup(MATH_KEYS, d.controls.weights, DEFAULT_MATH_WEIGHTS);
                rw = normalizeGroup(READ_KEYS,  d.controls.weights, DEFAULT_READ_WEIGHTS);
              } else if (d.controls.enabled) {
                const raw = {}; ALL_KEYS.forEach((k) => { raw[k] = d.controls.enabled[k] ? 50 : 0; });
                mw = normalizeGroup(MATH_KEYS, raw, DEFAULT_MATH_WEIGHTS);
                rw = normalizeGroup(READ_KEYS,  raw, DEFAULT_READ_WEIGHTS);
              } else { mw = DEFAULT_MATH_WEIGHTS; rw = DEFAULT_READ_WEIGHTS; }
              setControls({ totalDays: d.controls.totalDays ?? DEFAULT_CONTROLS.totalDays, mathPct: d.controls.mathPct ?? DEFAULT_CONTROLS.mathPct, mathWeights: mw, readWeights: rw });
            }
            if (d.completed) setCompleted(d.completed);
            if (d.openWeeks) setOpenWeeks(d.openWeeks);
            if (d.openDays)  setOpenDays(d.openDays);
          }
        }
      } catch {} finally { setLoaded(true); }
    })();
  }, []);

  useEffect(() => {
    if (!loaded) return;
    (async () => {
      try {
        if (typeof window !== 'undefined' && window.storage) {
          await window.storage.set(STORAGE_KEY, JSON.stringify({ controls, completed, openWeeks, openDays }));
        }
      } catch {}
    })();
  }, [controls, completed, openWeeks, openDays, loaded]);

  const plan = useMemo(() => buildPlan(controls), [controls]);

  const stats = useMemo(() => {
    let totalTasks = 0, doneTasks = 0, totalDays = 0, doneDays = 0;
    plan.weeks.forEach((w) => w.days.forEach((d) => {
      totalDays++;
      const ids = d.tasks.map((_, i) => taskId(d.id, i));
      const done = ids.filter((id) => completed[id]).length;
      totalTasks += ids.length; doneTasks += done;
      if (done === ids.length && ids.length) doneDays++;
    }));
    return { totalTasks, doneTasks, totalDays, doneDays, pct: totalTasks ? (doneTasks / totalTasks) * 100 : 0 };
  }, [plan, completed]);

  const weekState = (week) => {
    let total = 0, done = 0, daysDone = 0;
    week.days.forEach((d) => {
      const ids = d.tasks.map((_, i) => taskId(d.id, i));
      const dn = ids.filter((id) => completed[id]).length;
      total += ids.length; done += dn;
      if (dn === ids.length && ids.length) daysDone++;
    });
    const s = done === total && total > 0 ? 'full' : done > 0 ? 'partial' : 'empty';
    return { total, done, daysTotal: week.days.length, daysDone, s };
  };
  const dayState = (day) => {
    const ids = day.tasks.map((_, i) => taskId(day.id, i));
    const d = ids.filter((id) => completed[id]).length;
    const s = d === ids.length && ids.length ? 'full' : d > 0 ? 'partial' : 'empty';
    return { total: ids.length, done: d, s };
  };

  const toggleTask = (id) => setCompleted((c) => ({ ...c, [id]: !c[id] }));
  const toggleDay  = (day) => { const ids = day.tasks.map((_, i) => taskId(day.id, i)); const all = ids.every((id) => completed[id]); setCompleted((c) => { const n = { ...c }; ids.forEach((id) => { n[id] = !all; }); return n; }); };
  const toggleWeek = (week) => { const ids = []; week.days.forEach((d) => d.tasks.forEach((_, i) => ids.push(taskId(d.id, i)))); const all = ids.every((id) => completed[id]); setCompleted((c) => { const n = { ...c }; ids.forEach((id) => { n[id] = !all; }); return n; }); };
  const reset = () => { if (window.confirm('Reset all checked-off progress?')) setCompleted({}); };

  const setSectionMath = (key, v) => setControls((c) => ({ ...c, mathWeights: adjustGroup(MATH_KEYS, c.mathWeights, key, v) }));
  const setSectionRead = (key, v) => setControls((c) => ({ ...c, readWeights: adjustGroup(READ_KEYS, c.readWeights, key, v) }));

  const summary = useMemo(() => {
    if (!plan.counts) return 'Set at least one section above 0% to build a plan.';
    const c = plan.counts; const parts = [];
    if (c.mathContent) parts.push(`${c.mathContent} math`);
    if (c.readContent) parts.push(`${c.readContent} reading & writing`);
    if (c.diagnostic)  parts.push('1 diagnostic');
    if (c.testDays)    parts.push(`${c.testDays} test/review`);
    if (c.prep)        parts.push('1 final prep');
    return `${c.total}-day plan — ${parts.join(' · ')}`;
  }, [plan]);

  return (
    <>
      <style>{`
        @import url('https://fonts.googleapis.com/css2?family=Fraunces:opsz,wght@9..144,400;9..144,500;9..144,600;9..144,700&family=Geist:wght@300;400;500;600&family=JetBrains+Mono:wght@400;500&display=swap');
        :root { --bg:#f6f1e7;--bg-deep:#ebe3d2;--surface:#fbf8f1;--ink:#1a1410;--ink-soft:#4a3f33;--muted:#8a7e6e;--line:#e3dac6;--line-strong:#c9bda3;--accent:#7a1c2b; }
        .sat-root,.sat-root * { font-family:'Geist',-apple-system,system-ui,sans-serif; }
        .sat-root .font-display { font-family:'Fraunces',Georgia,serif; font-optical-sizing:auto; }
        .sat-root .font-mono    { font-family:'JetBrains Mono',ui-monospace,monospace; }
        .sat-collapse           { overflow:hidden; transition:grid-template-rows 240ms cubic-bezier(.4,0,.2,1); display:grid; }
        .sat-collapse-inner     { min-height:0; }
        .sat-task-done          { text-decoration:line-through; text-decoration-color:var(--line-strong); }
        .sat-chevron            { transition:transform 200ms ease; }
        .sat-chevron.open       { transform:rotate(180deg); }
        .sat-row-hover:hover    { background:var(--bg-deep); }
        .sat-range                          { -webkit-appearance:none; appearance:none; width:100%; height:4px; border-radius:999px; outline:none; }
        .sat-range::-webkit-slider-thumb   { -webkit-appearance:none; appearance:none; width:20px; height:20px; border-radius:50%; background:var(--surface); border:2px solid var(--accent); cursor:pointer; box-shadow:0 1px 3px rgba(0,0,0,.18); transition:transform 120ms; }
        .sat-range::-webkit-slider-thumb:hover { transform:scale(1.12); }
        .sat-range::-moz-range-thumb       { width:20px; height:20px; border-radius:50%; background:var(--surface); border:2px solid var(--accent); cursor:pointer; }
        .sat-range-sec::-webkit-slider-thumb { width:16px; height:16px; border-color:var(--sl); }
        .sat-range-sec::-moz-range-thumb     { width:16px; height:16px; border-color:var(--sl); }
      `}</style>

      <div className="sat-root min-h-screen w-full" style={{ background:'var(--bg)', color:'var(--ink)' }}>
        <div className="max-w-3xl mx-auto px-6 py-12 md:py-16">

          <div className="flex items-baseline justify-between mb-6">
            <h1 className="font-display font-semibold text-2xl tracking-tight">SAT Study Plan</h1>
            <span className="font-mono text-[10px] tracking-[0.2em] uppercase" style={{ color:'var(--muted)' }}>Builder · v6</span>
          </div>

          {/* CONTROL MENU */}
          <section className="mb-8 p-6 rounded-sm border" style={{ background:'var(--surface)', borderColor:'var(--line)' }}>
            <div className="font-mono text-[10px] tracking-[0.2em] uppercase mb-5" style={{ color:'var(--muted)' }}>Build your plan</div>
            <div className="mb-6">
              <div className="flex items-baseline justify-between mb-2">
                <label className="text-sm font-medium" style={{ color:'var(--ink-soft)' }}>Length</label>
                <span className="font-mono text-sm" style={{ color:'var(--ink)' }}>{controls.totalDays} days</span>
              </div>
              <input type="range" min={5} max={60} step={1} value={controls.totalDays}
                onChange={(e) => setControls((c) => ({ ...c, totalDays: Number(e.target.value) }))} className="sat-range"
                style={{ background: `linear-gradient(to right, var(--accent) 0%, var(--accent) ${((controls.totalDays-5)/55)*100}%, var(--line-strong) ${((controls.totalDays-5)/55)*100}%, var(--line-strong) 100%)` }} />
              <div className="flex justify-between font-mono text-[10px] mt-1" style={{ color:'var(--muted)' }}><span>5</span><span>60</span></div>
            </div>
            <div className="mb-8">
              <div className="flex items-baseline justify-between mb-2">
                <label className="text-sm font-medium" style={{ color:'var(--ink-soft)' }}>Focus · reading vs math</label>
                <span className="font-mono text-sm" style={{ color:'var(--ink)' }}>{controls.mathPct}% math · {100-controls.mathPct}% reading</span>
              </div>
              <input type="range" min={0} max={100} step={5} value={controls.mathPct}
                onChange={(e) => setControls((c) => ({ ...c, mathPct: Number(e.target.value) }))} className="sat-range"
                style={{ background: `linear-gradient(to right, var(--line-strong) 0%, var(--line-strong) ${100-controls.mathPct}%, var(--accent) ${100-controls.mathPct}%, var(--accent) 100%)` }} />
              <div className="flex justify-between font-mono text-[10px] mt-1" style={{ color:'var(--muted)' }}><span>All reading</span><span>All math</span></div>
            </div>
            <div className="grid sm:grid-cols-2 gap-x-8 gap-y-7">
              <div>
                <div className="font-mono text-[10px] tracking-[0.18em] uppercase mb-3" style={{ color:'var(--muted)' }}>Math sections · % of math days</div>
                <div className="flex flex-col gap-5">
                  {MATH_KEYS.map((k) => <SectionSlider key={k} value={controls.mathWeights[k]} label={DOMAINS[k].label} focus={DOMAINS[k].pillFocus} onChange={(v) => setSectionMath(k, v)} />)}
                </div>
              </div>
              <div>
                <div className="font-mono text-[10px] tracking-[0.18em] uppercase mb-3" style={{ color:'var(--muted)' }}>Reading & Writing · % of reading days</div>
                <div className="flex flex-col gap-5">
                  {READ_KEYS.map((k) => <SectionSlider key={k} value={controls.readWeights[k]} label={DOMAINS[k].label} focus={DOMAINS[k].pillFocus} onChange={(v) => setSectionRead(k, v)} />)}
                </div>
              </div>
            </div>
            <p className="mt-6 pt-4 text-[13px] font-mono border-t" style={{ borderColor:'var(--line)', color:'var(--ink-soft)' }}>→ {summary}</p>
          </section>

          {/* PROGRESS */}
          {!plan.empty && (
            <section className="mb-10 p-6 rounded-sm border" style={{ background:'var(--surface)', borderColor:'var(--line)' }}>
              <div className="flex items-end justify-between mb-4">
                <div>
                  <div className="font-mono text-[10px] tracking-[0.2em] uppercase mb-1" style={{ color:'var(--muted)' }}>Progress</div>
                  <div className="flex items-baseline gap-3">
                    <span className="font-display font-semibold" style={{ fontSize:'3rem', lineHeight:1 }}>{Math.round(stats.pct)}</span>
                    <span className="font-display text-2xl" style={{ color:'var(--muted)' }}>%</span>
                  </div>
                </div>
                <button onClick={reset} className="flex items-center gap-1.5 text-xs font-mono uppercase tracking-wider px-3 py-1.5 rounded-sm border transition-colors"
                  style={{ borderColor:'var(--line-strong)', color:'var(--ink-soft)' }}
                  onMouseEnter={(e) => e.currentTarget.style.background='var(--bg-deep)'} onMouseLeave={(e) => e.currentTarget.style.background='transparent'}>
                  <RotateCcw className="w-3 h-3" /> Reset
                </button>
              </div>
              <div className="relative h-2 rounded-full mb-4 overflow-hidden" style={{ background:'var(--bg-deep)' }}>
                <div className="absolute inset-y-0 left-0 rounded-full transition-all duration-500 ease-out" style={{ width:`${stats.pct}%`, background:'var(--accent)' }} />
              </div>
              <div className="flex flex-wrap gap-x-6 gap-y-1 font-mono text-xs" style={{ color:'var(--ink-soft)' }}>
                <span><span style={{ color:'var(--ink)' }}>{stats.doneTasks}</span><span style={{ color:'var(--muted)' }}> / {stats.totalTasks} tasks</span></span>
                <span><span style={{ color:'var(--ink)' }}>{stats.doneDays}</span><span style={{ color:'var(--muted)' }}> / {stats.totalDays} days complete</span></span>
              </div>
            </section>
          )}

          {plan.empty && (
            <div className="mb-10 p-10 rounded-sm border text-center" style={{ background:'var(--surface)', borderColor:'var(--line)' }}>
              <p className="font-display text-xl mb-1">No sections selected</p>
              <p className="text-sm" style={{ color:'var(--muted)' }}>Raise at least one section slider above 0% to generate your plan.</p>
            </div>
          )}

          {/* WEEKS */}
          <div className="space-y-4">
            {plan.weeks.map((week) => {
              const ws = weekState(week); const isOpen = !!openWeeks[week.id];
              return (
                <section key={week.id} className="rounded-sm border overflow-hidden" style={{ background:'var(--surface)', borderColor:'var(--line)' }}>
                  <div onClick={() => setOpenWeeks((s) => ({ ...s, [week.id]: !s[week.id] }))} className="cursor-pointer px-5 md:px-6 py-5 flex items-start gap-4 sat-row-hover transition-colors">
                    <CheckBox state={ws.s} onClick={() => toggleWeek(week)} size="lg" />
                    <div className="flex-1 min-w-0">
                      <div className="flex items-baseline gap-3 flex-wrap">
                        <span className="font-mono text-[10px] tracking-[0.2em] uppercase" style={{ color:'var(--muted)' }}>Week {week.weekNum}</span>
                        <span className="font-mono text-[10px]" style={{ color:'var(--line-strong)' }}>·</span>
                        <span className="font-mono text-[10px]" style={{ color:'var(--muted)' }}>{ws.daysDone}/{ws.daysTotal} days</span>
                      </div>
                      <h2 className="font-display font-medium text-xl md:text-2xl leading-tight mt-1">{week.title}</h2>
                    </div>
                    <ChevronDown className={`sat-chevron mt-1 w-5 h-5 ${isOpen?'open':''}`} style={{ color:'var(--muted)' }} />
                  </div>
                  <div className="sat-collapse" style={{ gridTemplateRows: isOpen ? '1fr' : '0fr' }}>
                    <div className="sat-collapse-inner">
                      <div style={{ borderTop:'1px solid var(--line)' }}>
                        {week.days.map((day, di) => {
                          const ds = dayState(day); const dayOpen = !!openDays[day.id]; const f = focusStyles[day.focus] || focusStyles.Review;
                          return (
                            <div key={day.id} style={{ borderTop: di === 0 ? 'none' : '1px solid var(--line)' }}>
                              <div onClick={() => setOpenDays((s) => ({ ...s, [day.id]: !s[day.id] }))} className="cursor-pointer px-5 md:px-6 py-3.5 flex items-center gap-4 sat-row-hover transition-colors">
                                <CheckBox state={ds.s} onClick={() => toggleDay(day)} size="md" />
                                <div className="flex-1 min-w-0">
                                  <div className="flex items-center gap-3 flex-wrap">
                                    <span className="font-mono text-[10px] tracking-[0.15em] uppercase" style={{ color:'var(--muted)' }}>{day.day}</span>
                                    <span className="font-mono text-[9px] tracking-[0.1em] uppercase px-1.5 py-0.5 rounded-sm" style={{ background:f.bg, color:f.fg }}>{day.focus}</span>
                                  </div>
                                  <h3 className={`font-display text-lg leading-snug mt-0.5 ${ds.s==='full'?'sat-task-done':''}`} style={{ color: ds.s==='full' ? 'var(--muted)' : 'var(--ink)' }}>{day.title}</h3>
                                </div>
                                <span className="font-mono text-xs hidden sm:inline" style={{ color:'var(--muted)' }}>{ds.done}/{ds.total}</span>
                                <ChevronDown className={`sat-chevron w-4 h-4 ${dayOpen?'open':''}`} style={{ color:'var(--muted)' }} />
                              </div>
                              <div className="sat-collapse" style={{ gridTemplateRows: dayOpen ? '1fr' : '0fr' }}>
                                <div className="sat-collapse-inner">
                                  <DayDetail day={day} />
                                </div>
                              </div>
                            </div>
                          );
                        })}
                      </div>
                    </div>
                  </div>
                </section>
              );
            })}
          </div>

          <footer className="mt-12 pt-6 border-t font-mono text-[10px] tracking-[0.15em] uppercase text-center" style={{ borderColor:'var(--line)', color:'var(--muted)' }}>
            Settings & progress save automatically · links verified May 2026
          </footer>
        </div>
      </div>
    </>
  );
}
</body>
</html>
