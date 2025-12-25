// ============================================================
// GLOBAL CONSTANTS & HELPERS
// ============================================================

// Rough "types" of people
const BELIEF_TYPES = {
  BELIEVER: "BELIEVER",
  WRONGDOER: "WRONGDOER",
  MIXED: "MIXED", // for in-between cases
};

// Kinds of life events
const EVENT_TYPES = {
  PROSPERITY: "PROSPERITY",
  POVERTY: "POVERTY",
  TEMPTATION: "TEMPTATION",
  TEST: "TEST", // generic hardship
};

// Event outcomes (reaction)
const OUTCOMES = {
  POSITIVE: "POSITIVE", // patience, gratitude, charity
  NEGATIVE: "NEGATIVE", // oppression, injustice, arrogance
};

// Simple helper functions
function clamp(value, min, max) {
  return Math.max(min, Math.min(max, value));
}

function randomInt(min, max) {
  // inclusive
  return Math.floor(Math.random() * (max - min + 1)) + min;
}

function randomChoice(list) {
  return list[randomInt(0, list.length - 1)];
}

// Map [inMin, inMax] -> [outMin, outMax]
function mapRange(value, inMin, inMax, outMin, outMax) {
  const t = (value - inMin) / (inMax - inMin);
  return outMin + t * (outMax - outMin);
}

// Moral score (0–100) → color from red (0) → yellow (50) → green (100)
function moralScoreToColor(score) {
  const s = clamp(score, 0, 100);
  const hue = mapRange(s, 0, 100, 0, 120); // 0=red, 120=green
  return `hsl(${hue}, 80%, 50%)`;
}

// ============================================================
// EVENT CLASS (one life event)
// ============================================================

class Event {
  constructor({ type, outcome, description, year, intensity = 1 }) {
    this.type = type;               // from EVENT_TYPES
    this.outcome = outcome;         // from OUTCOMES
    this.description = description; // human-readable text
    this.year = year;               // relative year in the story
    this.intensity = intensity;     // how strong the event is (1–3 for example)
  }
}

// ============================================================
// STORY CLASS (timeline of one person)
// ============================================================

class Story {
  constructor(person) {
    this.person = person;
    this.events = [];
  }

  addEvent(event) {
    this.events.push(event);
  }

  // Generate a sequence of random events over "years"
  generateRandomTimeline(years = 20) {
    for (let year = 1; year <= years; year++) {
      const eventType = randomChoice([
        EVENT_TYPES.PROSPERITY,
        EVENT_TYPES.POVERTY,
        EVENT_TYPES.TEMPTATION,
        EVENT_TYPES.TEST,
      ]);

      const event = this.person.experienceEvent(eventType, { year });
      this.addEvent(event);
    }
    return this;
  }

  // Simple text summary
  summarize() {
    return this.events.map((e) => {
      return `[Year ${e.year}] (${e.type} / ${e.outcome}) ${e.description}`;
    });
  }
}

// ============================================================
// PERSON CLASS
// ============================================================

class Person {
  constructor({
    id,
    name,
    beliefType = BELIEF_TYPES.MIXED,
    behaviour = "neutral",     // text label ("grateful", "oppressive", etc.)
    isGood = 50,               // 0-100
    parent = null,             // parent Person (good/bad tree)
    initialNotes = "",
  }) {
    this.id = id;
    this.name = name;
    this.beliefType = beliefType;
    this.behaviour = behaviour;
    this.isGood = clamp(isGood, 0, 100);
    this.parent = parent;
    this.initialNotes = initialNotes;
    this.events = [];
  }

  // ---- "Parent of itself" idea: create child Person with this as parent ----
  createChild(overrides = {}) {
    // Child’s goodness is influenced by parent, but can vary
    const drift = randomInt(-30, 30); // could be "good fruit from bad tree" etc.
    const childIsGood = clamp(this.isGood + drift, 0, 100);

    const childBeliefType =
      childIsGood > 70
        ? BELIEF_TYPES.BELIEVER
        : childIsGood < 30
        ? BELIEF_TYPES.WRONGDOER
        : BELIEF_TYPES.MIXED;

    return new Person({
      id: `${this.id}-child-${Date.now()}-${Math.random()
        .toString(16)
        .slice(2, 6)}`,
      name: `${this.name}'s child`,
      beliefType: childBeliefType,
      behaviour: childIsGood >= 50 ? "mostly_good" : "mostly_bad",
      isGood: childIsGood,
      parent: this,
      ...overrides,
    });
  }

  // Decide how this person reacts to an event, based on belief & isGood
  _decideOutcome(eventType) {
    // Base probability of positive outcome from isGood
    const basePositiveChance = this.isGood / 100; // 0–1

    // Adjustment based on belief type
    let modifier = 0;
    if (this.beliefType === BELIEF_TYPES.BELIEVER) {
      modifier += 0.2; // more likely to be patient & grateful
    } else if (this.beliefType === BELIEF_TYPES.WRONGDOER) {
      modifier -= 0.2; // more likely to be oppressive
    }

    // Certain events are harder (temptation) or easier
    if (eventType === EVENT_TYPES.TEMPTATION) {
      modifier -= 0.1; // temptation is tricky even for good people
    } else if (eventType === EVENT_TYPES.PROSPERITY) {
      // wealth can corrupt wrongdoer
      modifier += this.beliefType === BELIEF_TYPES.WRONGDOER ? -0.1 : 0.05;
    }

    const finalChance = clamp(basePositiveChance + modifier, 0, 1);

    return Math.random() < finalChance ? OUTCOMES.POSITIVE : OUTCOMES.NEGATIVE;
  }

  // Creates one Event and updates the person slightly
  experienceEvent(eventType, { year = 0 } = {}) {
    const outcome = this._decideOutcome(eventType);

    // Simple narrative text depending on type/outcome
    const description = this._buildEventDescription(eventType, outcome);

    // Adjust isGood a bit based on reaction (you can tune these numbers)
    const delta =
      outcome === OUTCOMES.POSITIVE
        ? randomInt(0, 3) // doing good tends to increase goodness slowly
        : randomInt(-4, 0); // doing wrong decreases it more sharply

    this.isGood = clamp(this.isGood + delta, 0, 100);

    const event = new Event({
      type: eventType,
      outcome,
      description,
      year,
      intensity: Math.abs(delta) || 1,
    });

    this.events.push(event);
    return event;
  }

  _buildEventDescription(eventType, outcome) {
    const base = `${this.name}`;

    if (eventType === EVENT_TYPES.PROSPERITY) {
      if (outcome === OUTCOMES.POSITIVE) {
        return `${base} received wealth and success, and responded with gratitude and generosity.`;
      } else {
        return `${base} received wealth and success, but became arrogant and hurtful.`;
      }
    }

    if (eventType === EVENT_TYPES.POVERTY) {
      if (outcome === OUTCOMES.POSITIVE) {
        return `${base} faced poverty with patience and trust.`;
      } else {
        return `${base} faced poverty with anger and oppression toward others.`;
      }
    }

    if (eventType === EVENT_TYPES.TEMPTATION) {
      if (outcome === OUTCOMES.POSITIVE) {
        return `${base} was tempted by forbidden pleasures but resisted.`;
      } else {
        return `${base} fell into forbidden pleasures and normalized them.`;
      }
    }

    if (eventType === EVENT_TYPES.TEST) {
      if (outcome === OUTCOMES.POSITIVE) {
        return `${base} went through a hard trial and grew closer to God and people.`;
      } else {
        return `${base} went through a hard trial and responded with injustice and resentment.`;
      }
    }

    return `${base} experienced an undefined event.`;
  }

  // Color representing this person on the “spectre”
  getMoralColor() {
    return moralScoreToColor(this.isGood);
  }

  // Small summary of the person
  describe() {
    return {
      id: this.id,
      name: this.name,
      beliefType: this.beliefType,
      behaviour: this.behaviour,
      isGood: this.isGood,
      color: this.getMoralColor(),
      parentName: this.parent ? this.parent.name : null,
    };
  }
}

// ============================================================
// COMMUNITY HELPERS (gradient spectrum of many people)
// ============================================================

function buildCommunitySpectrum(people) {
  // Sort people by isGood (0 → 100)
  const sorted = [...people].sort((a, b) => a.isGood - b.isGood);

  return sorted.map((person, index) => {
    const t =
      sorted.length === 1 ? 0 : index / (sorted.length - 1); // 0..1 position
    return {
      person: person.describe(),
      stop: t,             // position on gradient (0 = worst, 1 = best)
      color: person.getMoralColor(),
    };
  });
}

// Optionally: build a CSS gradient string to visualize the spectre
function communityGradientCSS(people) {
  const stops = buildCommunitySpectrum(people);
  const parts = stops.map((s) => {
    const percentage = Math.round(s.stop * 100);
    return `${s.color} ${percentage}%`;
  });
  return `linear-gradient(90deg, ${parts.join(", ")})`;
}

// ============================================================
// EXAMPLE USAGE
// ============================================================

// Initial people (good tree, bad tree, etc.)
const goodParent = new Person({
  id: "good-parent-1",
  name: "Aisha",
  beliefType: BELIEF_TYPES.BELIEVER,
  behaviour: "grateful",
  isGood: 90,
  initialNotes: "Good fruit from good tree candidate",
});

const badParent = new Person({
  id: "bad-parent-1",
  name: "Karim",
  beliefType: BELIEF_TYPES.WRONGDOER,
  behaviour: "oppressive",
  isGood: 15,
  initialNotes: "Bad fruit from bad tree candidate",
});

// Child examples
const goodFruitFromBadTree = badParent.createChild({
  name: "Yusuf",
  isGood: 85, // explicitly good despite bad parent
  beliefType: BELIEF_TYPES.BELIEVER,
});

const badFruitFromGoodTree = goodParent.createChild({
  name: "Lina",
  isGood: 20, // explicitly bad despite good parent
  beliefType: BELIEF_TYPES.WRONGDOER,
});

// Generate stories
const storyAisha = new Story(goodParent).generateRandomTimeline(15);
const storyKarim = new Story(badParent).generateRandomTimeline(15);
const storyYusuf = new Story(goodFruitFromBadTree).generateRandomTimeline(15);
const storyLina = new Story(badFruitFromGoodTree).generateRandomTimeline(15);

// Build gradient for all four
const community = [goodParent, badParent, goodFruitFromBadTree, badFruitFromGoodTree];
const spectrum = buildCommunitySpectrum(community);
const gradientCSS = communityGradientCSS(community);

// You can log these to see the result:
console.log("Community spectrum:", spectrum);
console.log("CSS gradient:", gradientCSS);
console.log("Story of Aisha:", storyAisha.summarize());
console.log("Story of Karim:", storyKarim.summarize());
