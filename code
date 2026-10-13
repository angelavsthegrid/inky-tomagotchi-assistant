// InkBlob.swift — a tiny, rude desktop ink-blob pet for macOS.
//
// Build & run (needs Xcode Command Line Tools — run `xcode-select --install` once):
//   swiftc -O InkBlob.swift -o InkBlob && ./InkBlob
//
// Drag it around. Click to poke it (poke it a lot and find out). Right-click for the menu:
// feed, morph, become…, spit at me, attitude (Sweet → Unhinged), sounds, talk, Gemini key, quit.
// Optional Gemini brain: right-click > "Gemini API key…" and paste a free key from aistudio.google.com,
// or launch with:  GEMINI_API_KEY=your_key ./InkBlob      (change model with GEMINI_MODEL=...)

import AppKit

// MARK: - Tweakables
let petName = "Inky"
let restSeconds = 8.0...20.0       // time spent as a plain blob between shapes
let holdSeconds = 3.5...7.0        // time spent holding a shape
let trickSeconds = 40.0...120.0    // random tricks: tongue, fly visits, spitting
let chatterMinutes = 6.0...12.0    // unprompted Gemini remarks (only when a key is set)
let defaultModel = "gemini-2.5-flash"

let shapeNames = ["star", "heart", "ghost", "cat", "mushroom", "bolt", "crown", "cactus", "poop", "skull",
                  "splat", "octopus", "finger", "tombstone", "dumpster", "toilet", "chicken"]

// MARK: - Attitude
enum Attitude: Int, CaseIterable {
    case sweet, snarky, mean, unhinged
    var title: String { ["Sweet", "Snarky", "Mean", "Unhinged (swears)"][rawValue] }
    var style: String {
        switch self {
        case .sweet: return "sweet, wholesome and encouraging. No swearing, no insults."
        case .snarky: return "sarcastic and snarky with dry wit and mild teasing. No swearing."
        case .mean: return "mean and condescending. Roast the human hard. Rude, but no swear words."
        case .unhinged: return "unhinged, foul-mouthed and savage. Swear freely (fuck, shit, ass, damn, etc.) and roast the human brutally but playfully. Never use slurs or attack real groups of people."
        }
    }
    static var current: Attitude {
        get { Attitude(rawValue: UserDefaults.standard.object(forKey: "attitude") as? Int ?? 1) ?? .snarky }
        set { UserDefaults.standard.set(newValue.rawValue, forKey: "attitude") }
    }
}

func pick(_ table: [Attitude: [String]]) -> String {
    (table[Attitude.current] ?? table[.snarky] ?? ["blorp."]).randomElement()!
}

// MARK: - Lines
let quips: [String: [String]] = [
    "blob": ["blorp.", "just vibing.", "poke me again. I dare you."],
    "star": ["I'm kind of a big deal.", "✨ main character energy ✨"],
    "heart": ["ew, feelings.", "don't make it weird."],
    "ghost": ["boo. pay your rent.", "I died of boredom."],
    "cat": ["meow. that's all you get.", "knock your coffee over? say less."],
    "mushroom": ["I'm a fun guy.", "spore-adic behavior."],
    "bolt": ["ZAP. feel productive yet?", "I run on spite."],
    "crown": ["kneel, peasant.", "royal ink, actually."],
    "cactus": ["don't touch me.", "personal space: 6 feet."],
    "poop": ["I am exactly what you think I am.", "this is your desktop now."],
    "skull": ["deadline vibes.", "I'm dead. you're next."],
    "splat": ["oops.", "that was on purpose."],
    "octopus": ["8 arms, 0 jobs.", "tentacle time."],
    "finger": ["no notes.", "my review of your day."],
    "tombstone": ["it was the meetings.", "gone but not forgotten. mostly forgotten."],
    "dumpster": ["this is fine. 🔥", "a self-portrait."],
    "toilet": ["take a seat.", "where your ideas belong."],
    "chicken": ["*squeak*", "bawk bawk, motherclucker."],
]
let profaneQuips: [String: [String]] = [
    "finger": ["fuck you specifically.", "this one's for your boss.", "sit on it and spin."],
    "poop": ["holy shit. literally.", "I'm the shit. you're just shit."],
    "dumpster": ["this is your week. it's on fucking fire.", "everything's fine. 🔥 everything's fucked."],
    "toilet": ["your ideas, as furniture.", "flush your expectations, dipshit."],
    "chicken": ["bawk. fuck.", "*squeaks aggressively*"],
    "skull": ["dead inside. same as you.", "I died and it was still better than your Monday."],
    "tombstone": ["here lies your will to work. good riddance."],
    "cactus": ["touch me and get fucked up."],
]
let idleLines: [Attitude: [String]] = [
    .sweet: ["you're doing great!", "proud of you!", "drink some water 💧", "you've got this!"],
    .mean: ["your posture is a war crime.", "close a tab. any tab.", "I've seen your browser history.",
            "that email can wait. your dignity can't.", "is this really what you're doing today?"],
    .unhinged: ["get your shit together.", "who the fuck approved this wallpaper.", "hydrate, you crusty bastard.",
                "this desktop is a goddamn crime scene.", "I'm not mad. I'm ink. there's a difference, asshole.",
                "do one productive thing. one. holy shit."],
]
let pokeLines: [Attitude: [String]] = [
    .sweet: ["hehe!", "boop!", "hi friend!", "that tickles :)"],
    .snarky: ["hey!", "rude.", "blorp?", "personal space!", "do it again and see what happens."],
    .mean: ["touch me again and I'll stain your keyboard.", "do you not have a job?", "wow. riveting.", "back off, meatbag."],
    .unhinged: ["what the fuck do you want.", "poke me one more time, dipshit.", "hands off, you absolute walnut.",
                "I will ink your entire ass.", "oh great, it's you again."],
]
let hungryLines: [Attitude: [String]] = [
    .sweet: ["I'm a little hungry 🥺"],
    .snarky: ["feed me ink…", "I'm wasting away here.", "hungry. right-click. you know what to do."],
    .mean: ["I'm starving and it's your fault.", "neglect. classic you.", "even your plants get fed more."],
    .unhinged: ["feed me or I swear to god.", "FEED. ME. you useless sack of meat.", "I'm hungry as shit. do something."],
]
let feedLines: [Attitude: [String]] = [
    .sweet: ["yummy! thank you!", "nom nom 💖"],
    .snarky: ["nom nom ink 🖋️", "delicious. more.", "*burp*"],
    .mean: ["finally. took you long enough.", "adequate."],
    .unhinged: ["about damn time.", "mm. tastes like your bad decisions.", "fine. you can live."],
]
let rageLines: [Attitude: [String]] = [
    .sweet: ["ouch! okay okay!"],
    .snarky: ["ENOUGH.", "you're poking a liquid. think about that."],
    .mean: ["that's it. you're getting inked.", "assault. with a cursor."],
    .unhinged: ["THAT'S IT, MOTHERFUCKER.", "you poked the wrong goddamn blob.", "eat ink, asshole."],
]
let spitLines: [Attitude: [String]] = [
    .sweet: ["oops, sneeze! 🤧"],
    .snarky: ["ptoo.", "say it, don't spray it. I chose spray."],
    .mean: ["that's for your playlist.", "aim assist: on."],
    .unhinged: ["ptoo. right in your stupid cursor.", "spit on that shit."],
]
let dodgeLines: [Attitude: [String]] = [
    .mean: ["don't touch me.", "personal space, creep.", "too slow."],
    .unhinged: ["get your grubby cursor off me.", "nice try, dumbass.", "fuck off, I'm busy."],
]
let wakeLines: [Attitude: [String]] = [
    .sweet: ["*yawn* hi!"],
    .snarky: ["it's literally night. why.", "I was dreaming of a cleaner desktop."],
    .mean: ["I was ASLEEP.", "go to bed, you goblin."],
    .unhinged: ["what the fuck, I was sleeping.", "it's late as shit. go to bed."],
]
let blepLines = ["blep.", "mlem.", "😛"]
let flyLines = ["bzz.", "get it off me.", "a fly? on MY poop?"]

// MARK: - Geometry
typealias Poly = [CGPoint]
let N = 180   // every shape is resampled to this many points so any shape can morph into any other

func P(_ x: CGFloat, _ y: CGFloat) -> CGPoint { CGPoint(x: x, y: y) }
func lerp(_ a: CGFloat, _ b: CGFloat, _ t: CGFloat) -> CGFloat { a + (b - a) * t }
func lerpP(_ a: CGPoint, _ b: CGPoint, _ t: CGFloat) -> CGPoint { P(lerp(a.x, b.x, t), lerp(a.y, b.y, t)) }

func easeOutBack(_ t: CGFloat) -> CGFloat {
    let c1: CGFloat = 1.4, c3 = c1 + 1, u = t - 1
    return 1 + c3 * u * u * u + c1 * u * u
}

func chaikin(_ p: Poly, _ iterations: Int) -> Poly {
    var pts = p
    for _ in 0..<iterations {
        var out: Poly = []
        for i in 0..<pts.count {
            let a = pts[i], b = pts[(i + 1) % pts.count]
            out.append(lerpP(a, b, 0.25)); out.append(lerpP(a, b, 0.75))
        }
        pts = out
    }
    return pts
}

func polar(steps: Int = 360, _ f: (CGFloat) -> CGFloat) -> Poly {
    (0..<steps).map { i in
        let t = CGFloat(i) / CGFloat(steps) * 2 * .pi
        let r = f(t)
        return P(cos(t) * r, sin(t) * r)
    }
}

func ellipseWidth(_ y: CGFloat, c: CGFloat, h: CGFloat, w: CGFloat, round: Bool = false) -> CGFloat {
    let d = abs((y - c) / h)
    guard d < 1 else { return 0 }
    return round ? w * sqrt(1 - d * d) : w * pow(1 - d * d * d * d, 0.25)
}

func profile(from y0: CGFloat, to y1: CGFloat, steps: Int = 160, _ w: (CGFloat) -> CGFloat) -> Poly {
    var right: Poly = [], left: Poly = []
    for i in 0...steps {
        let y = lerp(y0, y1, CGFloat(i) / CGFloat(steps))
        let x = max(0.02, w(y))
        right.append(P(x, y)); left.append(P(-x, y))
    }
    return right + left.reversed()
}

func resample(_ pts: Poly, count: Int = N) -> Poly {
    let closed = pts + [pts[0]]
    var lens: [CGFloat] = [0]
    for i in 1..<closed.count {
        lens.append(lens[i - 1] + hypot(closed[i].x - closed[i - 1].x, closed[i].y - closed[i - 1].y))
    }
    let total = lens.last!
    var out: Poly = []
    var j = 1
    for k in 0..<count {
        let d = total * CGFloat(k) / CGFloat(count)
        while j < lens.count - 1 && lens[j] < d { j += 1 }
        let seg = lens[j] - lens[j - 1]
        let t = seg > 0 ? (d - lens[j - 1]) / seg : 0
        out.append(lerpP(closed[j - 1], closed[j], t))
    }
    return out
}

/// Same winding, fit into [-1, 1], resampled, starting from the top-center point (keeps morphs from twisting).
func normalize(_ raw: Poly) -> (Poly, CGFloat, CGFloat, CGFloat) {
    var p = raw
    var area: CGFloat = 0
    for i in 0..<p.count { let a = p[i], b = p[(i + 1) % p.count]; area += a.x * b.y - b.x * a.y }
    if area < 0 { p.reverse() }
    let xs = p.map(\.x), ys = p.map(\.y)
    let cx = (xs.min()! + xs.max()!) / 2, cy = (ys.min()! + ys.max()!) / 2
    let s = max(xs.max()! - xs.min()!, ys.max()! - ys.min()!) / 2
    p = resample(p.map { P(($0.x - cx) / s, ($0.y - cy) / s) })
    var best = 0, bestD = CGFloat.infinity
    for (i, q) in p.enumerated() where q.y > 0 && abs(q.x) < bestD { bestD = abs(q.x); best = i }
    return (Array(p[best...] + p[..<best]), cx, cy, s)
}

func inside(_ q: CGPoint, _ p: Poly) -> Bool {
    var c = false
    var j = p.count - 1
    for i in 0..<p.count {
        let a = p[i], b = p[j]
        if (a.y > q.y) != (b.y > q.y) && q.x < (b.x - a.x) * (q.y - a.y) / (b.y - a.y) + a.x { c.toggle() }
        j = i
    }
    return c
}

/// Roomiest spot inside the shape (slightly biased upward) — that's where the face goes.
func faceSpot(_ p: Poly) -> (CGPoint, CGFloat) {
    var best = CGPoint.zero, bestScore: CGFloat = -1, bestD: CGFloat = 0
    for gx in 0...40 {
        for gy in 0...40 {
            let q = P(-1 + CGFloat(gx) / 20, -1 + CGFloat(gy) / 20)
            guard inside(q, p) else { continue }
            let d = p.map { hypot($0.x - q.x, $0.y - q.y) }.min()!
            let score = d + 0.1 * q.y
            if score > bestScore { bestScore = score; best = q; bestD = d }
        }
    }
    return (best, bestD)
}

// MARK: - Shapes
struct Shape {
    let name: String
    let pts: Poly
    let eye: CGPoint
    let eyeR: CGFloat
    var cx: CGFloat = 0, cy: CGFloat = 0, s: CGFloat = 1
    /// Raw shape coordinates → normalized unit space (for decorations like fire and text).
    func norm(_ p: CGPoint) -> CGPoint { P((p.x - cx) / s, (p.y - cy) / s) }
}

func rawShape(_ name: String) -> Poly {
    switch name {
    case "star":
        return chaikin((0..<10).map { k in
            let a = CGFloat.pi / 2 + CGFloat(k) * .pi / 5
            let r: CGFloat = k % 2 == 0 ? 1 : 0.45
            return P(cos(a) * r, sin(a) * r)
        }, 1)
    case "heart":
        return (0..<240).map { i in
            let t = CGFloat(i) / 240 * 2 * .pi, s = sin(t)
            return P(16 * s * s * s, 13 * cos(t) - 5 * cos(2 * t) - 2 * cos(3 * t) - cos(4 * t))
        }
    case "ghost":
        var p: Poly = []
        for i in 0...60 { let a = CGFloat(i) / 60 * .pi; p.append(P(0.8 * cos(a), 0.2 + 0.8 * sin(a))) }
        p.append(P(-0.8, -0.7))
        for i in 0...60 {
            let x = lerp(-0.8, 0.8, CGFloat(i) / 60)
            p.append(P(x, -0.85 + 0.15 * cos((x + 0.8) / 1.6 * 3 * 2 * .pi)))
        }
        p.append(P(0.8, -0.7))
        return p
    case "cat":
        return polar { t in
            var r: CGFloat = 0.75 + 0.05 * cos(2 * t)
            for e in [CGFloat.pi * 0.3, CGFloat.pi * 0.7] { r += 0.6 * max(0, 1 - abs(t - e) / 0.3) }
            return r
        }
    case "mushroom":
        var p: Poly = [P(0.3, -1), P(0.25, -0.05), P(0.95, -0.1)]
        for i in 0...60 { let a = CGFloat(i) / 60 * .pi; p.append(P(0.95 * cos(a), -0.05 + 0.85 * sin(a))) }
        p += [P(-0.95, -0.1), P(-0.25, -0.05), P(-0.3, -1)]
        return chaikin(p, 2)
    case "bolt":
        return chaikin([P(0.15, 1), P(-0.55, -0.05), P(-0.05, -0.05), P(-0.35, -1),
                        P(0.55, 0.15), P(0.05, 0.15), P(0.45, 1)], 1)
    case "crown":
        return chaikin([P(-1, -0.6), P(1, -0.6), P(1, 0.6), P(0.5, 0.05),
                        P(0, 0.8), P(-0.5, 0.05), P(-1, 0.6)], 1)
    case "cactus":
        return chaikin([P(-0.2, -1), P(0.2, -1), P(0.2, -0.2), P(0.6, -0.2), P(0.6, 0.45), P(0.4, 0.45),
                        P(0.4, 0.0), P(0.2, 0.0), P(0.2, 0.9), P(-0.2, 0.9), P(-0.2, 0.2), P(-0.4, 0.2),
                        P(-0.4, 0.6), P(-0.6, 0.6), P(-0.6, 0.0), P(-0.2, 0.0)], 2)
    case "poop":
        return chaikin(profile(from: -1, to: 0.98) { y in
            max(ellipseWidth(y, c: -0.68, h: 0.32, w: 1.0),
                ellipseWidth(y, c: -0.12, h: 0.28, w: 0.74),
                ellipseWidth(y, c: 0.36, h: 0.24, w: 0.48),
                (y > 0.5 && y < 0.98) ? 0.24 * (0.98 - y) / 0.48 : 0)
        }, 1)
    case "skull":
        return chaikin(profile(from: -1, to: 1) { y in
            max(ellipseWidth(y, c: 0.2, h: 0.8, w: 0.85, round: true),
                ellipseWidth(y, c: -0.62, h: 0.38, w: 0.5))
        }, 1)
    case "splat":
        let spikes = (0..<Int.random(in: 6...10)).map { _ in
            (CGFloat.random(in: 0...(2 * .pi)), CGFloat.random(in: 0.2...0.55))
        }
        return polar(steps: 400) { t in
            var r: CGFloat = 0.55
            for (a, len) in spikes {
                var d = abs(t - a); d = min(d, 2 * .pi - d)
                r += len * exp(-(d / 0.09) * (d / 0.09))
            }
            return r
        }
    case "octopus":
        return polar(steps: 400) { t in
            let s = sin(t)
            return s >= 0 ? 0.75 : 0.75 + 0.45 * abs(s) * pow(abs(sin(4 * t)), 1.5)
        }
    case "finger":
        return chaikin([P(-0.6, -1), P(0.55, -1), P(0.6, -0.1), P(0.6, 0.05), P(0.42, 0.1), P(0.4, 0.22),
                        P(0.15, 0.25), P(0.14, 1.0), P(-0.14, 1.0), P(-0.15, 0.25), P(-0.42, 0.22),
                        P(-0.45, 0.1), P(-0.62, 0.05), P(-0.65, -0.3)], 2)
    case "tombstone":
        var p: Poly = [P(-0.85, -1), P(0.85, -1), P(0.85, -0.82), P(0.62, -0.82), P(0.62, 0.35)]
        for i in 0...40 { let a = CGFloat(i) / 40 * .pi; p.append(P(0.62 * cos(a), 0.35 + 0.62 * sin(a))) }
        p += [P(-0.62, -0.82), P(-0.85, -0.82)]
        return chaikin(p, 1)
    case "dumpster":
        var p: Poly = [P(-0.8, -0.85), P(-0.7, -1), P(-0.55, -0.85), P(0.55, -0.85), P(0.7, -1), P(0.8, -0.85),
                       P(0.95, 0.15), P(1.0, 0.25)]
        let n = 9
        for i in 0...(n * 2) {   // flames, right to left
            let x = lerp(0.85, -0.85, CGFloat(i) / CGFloat(n * 2))
            p.append(P(x, i % 2 == 0 ? 0.3 : CGFloat.random(in: 0.6...1.0)))
        }
        p += [P(-1.0, 0.25), P(-0.95, 0.15)]
        return chaikin(p, 1)
    case "toilet":
        return chaikin([P(-0.25, -1), P(0.25, -1), P(0.3, -0.6), P(0.75, -0.35), P(0.92, -0.05), P(0.92, 0.08),
                        P(-0.45, 0.08), P(-0.45, 1.0), P(-0.9, 1.0), P(-0.9, 0.0), P(-0.6, -0.1), P(-0.4, -0.5)], 1)
    case "chicken":
        return chaikin([P(-1, 0.55), P(-0.84, 0.5), P(-0.8, 0.4), P(-0.84, 0.3), P(-0.74, 0.26), P(-0.6, 0.12),
                        P(-0.4, -0.08), P(-0.2, -0.22), P(0.05, -0.42), P(0.2, -0.48), P(0.22, -0.8), P(0.12, -0.88),
                        P(0.34, -0.88), P(0.3, -0.5), P(0.5, -0.5), P(0.8, -0.35), P(1.0, -0.05), P(0.97, 0.18),
                        P(0.86, 0.06), P(0.6, 0.1), P(0.3, 0.05), P(0.0, 0.0), P(-0.3, 0.15), P(-0.52, 0.4),
                        P(-0.6, 0.6), P(-0.58, 0.76), P(-0.65, 0.72), P(-0.68, 0.84), P(-0.73, 0.75), P(-0.79, 0.82),
                        P(-0.8, 0.68), P(-0.88, 0.62)], 1)
    default: // "blob" — a little different every time
        let a = CGFloat.random(in: 0.03...0.08), b = CGFloat.random(in: 0.02...0.06)
        let p1 = CGFloat.random(in: 0...6.28), p2 = CGFloat.random(in: 0...6.28), p3 = CGFloat.random(in: 0...6.28)
        return polar { t in 0.9 + a * sin(3 * t + p1) + b * sin(5 * t + p2) + 0.02 * sin(7 * t + p3) - 0.06 * sin(t) }
    }
}

func makeShape(_ name: String) -> Shape {
    let (p, cx, cy, s) = normalize(rawShape(name))
    let (c, d) = faceSpot(p)
    return Shape(name: name, pts: p, eye: c, eyeR: d, cx: cx, cy: cy, s: s)
}

// MARK: - Sounds
func sfx(_ name: String) {
    guard UserDefaults.standard.object(forKey: "sounds") as? Bool ?? true else { return }
    NSSound(named: NSSound.Name(name))?.play()
}

// MARK: - Ink spit overlay (click-through, covers your screens)
final class SplatOverlay: NSView {
    struct Drop { var from: CGPoint; var to: CGPoint; var t0: CFTimeInterval; var splat: Poly; var size: CGFloat; var landed = false }
    let flight: CFTimeInterval = 0.45
    let overlayWindow: NSWindow
    private var drops: [Drop] = []
    private var timer: Timer?

    init() {
        let frame = NSScreen.screens.reduce(NSRect.null) { $0.union($1.frame) }
        let w = NSWindow(contentRect: frame, styleMask: [.borderless], backing: .buffered, defer: false)
        w.isOpaque = false
        w.backgroundColor = .clear
        w.hasShadow = false
        w.ignoresMouseEvents = true
        w.level = .floating
        w.collectionBehavior = [.canJoinAllSpaces, .stationary, .ignoresCycle]
        overlayWindow = w
        super.init(frame: NSRect(origin: .zero, size: frame.size))
        w.contentView = self
        w.orderFrontRegardless()
    }
    required init?(coder: NSCoder) { fatalError() }
    override var isOpaque: Bool { false }

    /// Screen coordinates in, ink out.
    func spit(from: CGPoint, to: CGPoint, delay: Double) {
        let o = overlayWindow.frame.origin
        let splat = normalize(rawShape("splat")).0
        drops.append(Drop(from: P(from.x - o.x, from.y - o.y), to: P(to.x - o.x, to.y - o.y),
                          t0: CACurrentMediaTime() + delay, splat: splat, size: CGFloat.random(in: 12...22)))
        if timer == nil {
            let t = Timer(timeInterval: 1.0 / 60, repeats: true) { [weak self] _ in self?.tick() }
            RunLoop.main.add(t, forMode: .common)
            timer = t
        }
    }

    private func tick() {
        let now = CACurrentMediaTime()
        for i in drops.indices where !drops[i].landed && now - drops[i].t0 >= flight {
            drops[i].landed = true
            sfx("Pop")
        }
        drops.removeAll { now - $0.t0 > flight + 4.5 }
        needsDisplay = true
        if drops.isEmpty { timer?.invalidate(); timer = nil }
    }

    override func draw(_ dirtyRect: NSRect) {
        let now = CACurrentMediaTime()
        NSColor.black.setFill()
        for d in drops {
            let age = now - d.t0
            guard age >= 0 else { continue }
            if age < flight {
                let t = CGFloat(age / flight)
                let p = lerpP(d.from, d.to, t)
                let y = p.y + 90 * sin(.pi * t)
                NSBezierPath(ovalIn: NSRect(x: p.x - 4, y: y - 4, width: 8, height: 8)).fill()
            } else {
                let fade = age < flight + 2.5 ? 1 : max(0, 1 - (age - flight - 2.5) / 2)
                NSColor(white: 0, alpha: CGFloat(fade)).setFill()
                let path = NSBezierPath()
                for (i, q) in d.splat.enumerated() {
                    let pt = P(d.to.x + q.x * d.size, d.to.y + q.y * d.size)
                    if i == 0 { path.move(to: pt) } else { path.line(to: pt) }
                }
                path.close()
                path.fill()
            }
        }
    }
}

// MARK: - Gemini (optional)
enum Gemini {
    static var key: String? {
        if let k = UserDefaults.standard.string(forKey: "geminiKey"), !k.isEmpty { return k }
        if let k = ProcessInfo.processInfo.environment["GEMINI_API_KEY"], !k.isEmpty { return k }
        return nil
    }
    static var model: String { ProcessInfo.processInfo.environment["GEMINI_MODEL"] ?? defaultModel }

    static func ask(_ userText: String, mood: String,
                    completion: @escaping (_ say: String?, _ shape: String?, _ action: String?) -> Void) {
        guard let key = key,
              let url = URL(string: "https://generativelanguage.googleapis.com/v1beta/models/\(model):generateContent")
        else { completion(nil, nil, nil); return }
        let prompt = """
        You are \(petName), a little black ink blob living on someone's Mac desktop like a tamagotchi. \
        Personality: \(Attitude.current.style) \
        You can shapeshift into: blob, \(shapeNames.joined(separator: ", ")). \
        You can also do an action: none, spit (spit ink at their cursor), blep (stick your tongue out), fly (a fly buzzes around you), dodge (scoot away). \
        Current mood: \(mood). \
        Reply ONLY with JSON: {"say": "<max 20 words, funny, in character>", "shape": "<one shape name>", "action": "<one action>"}
        The human says: \(userText)
        """
        var req = URLRequest(url: url)
        req.httpMethod = "POST"
        req.setValue("application/json", forHTTPHeaderField: "Content-Type")
        req.setValue(key, forHTTPHeaderField: "x-goog-api-key")
        let body: [String: Any] = [
            "contents": [["parts": [["text": prompt]]]],
            "generationConfig": ["responseMimeType": "application/json", "temperature": 1.15],
            "safetySettings": [["category": "HARM_CATEGORY_HARASSMENT", "threshold": "BLOCK_ONLY_HIGH"]],
        ]
        req.httpBody = try? JSONSerialization.data(withJSONObject: body)
        URLSession.shared.dataTask(with: req) { data, _, err in
            var say: String?, shape: String?, action: String?
            if let data, let root = try? JSONSerialization.jsonObject(with: data) as? [String: Any] {
                if let cands = root["candidates"] as? [[String: Any]],
                   let content = cands.first?["content"] as? [String: Any],
                   let parts = content["parts"] as? [[String: Any]],
                   let text = parts.first?["text"] as? String {
                    if let d = text.data(using: .utf8),
                       let j = try? JSONSerialization.jsonObject(with: d) as? [String: Any] {
                        say = j["say"] as? String
                        shape = (j["shape"] as? String)?.lowercased()
                        action = (j["action"] as? String)?.lowercased()
                    } else {
                        say = text
                    }
                } else if let e = root["error"] as? [String: Any] {
                    say = "Gemini error: \(e["message"] as? String ?? "unknown")"
                } else {
                    say = "my brain refused to answer that one."
                }
            } else if let err {
                say = "can't reach my brain: \(err.localizedDescription)"
            }
            DispatchQueue.main.async { completion(say, shape, action) }
        }.resume()
    }
}

func prompt(_ title: String, _ message: String, secure: Bool = false) -> String? {
    NSApp.activate(ignoringOtherApps: true)
    let alert = NSAlert()
    alert.messageText = title
    alert.informativeText = message
    alert.addButton(withTitle: "OK")
    alert.addButton(withTitle: "Cancel")
    let frame = NSRect(x: 0, y: 0, width: 280, height: 24)
    let field: NSTextField = secure ? NSSecureTextField(frame: frame) : NSTextField(frame: frame)
    alert.accessoryView = field
    alert.window.initialFirstResponder = field
    return alert.runModal() == .alertFirstButtonReturn ? field.stringValue : nil
}

func drawCentered(_ text: String, at p: CGPoint, size: CGFloat, weight: NSFont.Weight, color: NSColor) {
    let s = NSAttributedString(string: text, attributes: [.font: NSFont.systemFont(ofSize: size, weight: weight),
                                                          .foregroundColor: color])
    let sz = s.size()
    s.draw(at: P(p.x - sz.width / 2, p.y - sz.height / 2))
}

// MARK: - The pet
final class BlobView: NSView {
    enum Mode { case resting, showing }

    let overlay: SplatOverlay
    private var cache: [String: Shape] = [:]
    private var from: Shape
    private var to: Shape
    private var base: Poly
    private var eyeC: CGPoint
    private var eyeR: CGFloat
    private var rendered: Poly = []
    private var sxy = P(1, 1)
    private var shakeX: CGFloat = 0
    private var morphStart: CFTimeInterval = 0
    private var morphDur: CFTimeInterval = 1
    private var morphT: CGFloat = 1

    private var mode = Mode.resting
    private var nextChange: CFTimeInterval
    private var nextWobble: CFTimeInterval
    private var nextBlink: CFTimeInterval
    private var nextNag: CFTimeInterval
    private var nextChatter: CFTimeInterval
    private var nextTrick: CFTimeInterval
    private var nextSleepCheck: CFTimeInterval = 0
    private var nextDodge: CFTimeInterval = 0
    private var lastInteraction: CFTimeInterval
    private var blinkUntil: CFTimeInterval = 0
    private var squishUntil: CFTimeInterval = 0
    private var happyUntil: CFTimeInterval = 0
    private var angryUntil: CFTimeInterval = 0
    private var shakeUntil: CFTimeInterval = 0
    private var blepUntil: CFTimeInterval = 0
    private var flyUntil: CFTimeInterval = 0
    private var bubbleHideAt: CFTimeInterval = 0
    private var sleeping = false
    private var mouseWasNear = false
    private var pokeTimes: [CFTimeInterval] = []
    private var flyPos = CGPoint.zero
    private var flyVisible = false

    private let bubble = NSView()
    private let label = NSTextField(wrappingLabelWithString: "")
    private var timer: Timer?
    private var dragStart: NSPoint?
    private var winStart = NSPoint.zero
    private var dragged = false

    private let R: CGFloat = 92
    private var center: CGPoint { P(bounds.midX, 112) }
    private var attitude: Attitude { Attitude.current }

    private var lastFed: Date {
        get { UserDefaults.standard.object(forKey: "lastFed") as? Date ?? Date() }
        set { UserDefaults.standard.set(newValue, forKey: "lastFed") }
    }
    private var hunger: Double { min(100, Date().timeIntervalSince(lastFed) / 3600 * 12) } // starving after ~8h
    private var hungry: Bool { hunger > 70 }

    init(frame: NSRect, overlay: SplatOverlay) {
        self.overlay = overlay
        let now = CACurrentMediaTime()
        let s = makeShape("blob")
        from = s; to = s; base = s.pts; eyeC = s.eye; eyeR = s.eyeR
        nextChange = now + 4; nextWobble = now + 2; nextBlink = now + 3; nextNag = now + 60
        nextTrick = now + Double.random(in: 20...40)
        nextChatter = now + 60 * Double.random(in: chatterMinutes)
        lastInteraction = now
        super.init(frame: frame)
        if UserDefaults.standard.object(forKey: "lastFed") == nil { lastFed = Date() }

        bubble.wantsLayer = true
        bubble.layer?.backgroundColor = NSColor.white.cgColor
        bubble.layer?.cornerRadius = 12
        bubble.layer?.borderColor = NSColor.black.cgColor
        bubble.layer?.borderWidth = 2
        label.font = .systemFont(ofSize: 12, weight: .semibold)
        label.textColor = .black
        label.isSelectable = false
        label.maximumNumberOfLines = 4
        bubble.addSubview(label)
        bubble.isHidden = true
        addSubview(bubble)

        let t = Timer(timeInterval: 1.0 / 60, repeats: true) { [weak self] _ in self?.tick() }
        RunLoop.main.add(t, forMode: .common)
        timer = t
        say("hi. I'm \(petName). I'm made of ink.")
    }
    required init?(coder: NSCoder) { fatalError() }
    override var isOpaque: Bool { false }
    override func acceptsFirstMouse(for event: NSEvent?) -> Bool { true }

    // MARK: Shapes & morphing
    private func shape(_ name: String) -> Shape {
        if name == "blob" || name == "splat" { return makeShape(name) }   // random every time
        if let s = cache[name] { return s }
        let s = makeShape(name); cache[name] = s; return s
    }

    private func randomShapeName() -> String {
        shapeNames.filter { $0 != to.name && !(attitude == .sweet && $0 == "finger") }.randomElement()!
    }

    private func startMorph(_ s: Shape, _ duration: Double) {
        from = Shape(name: to.name, pts: base, eye: eyeC, eyeR: eyeR, cx: to.cx, cy: to.cy, s: to.s)
        to = s
        morphStart = CACurrentMediaTime()
        morphDur = duration
    }

    func morph(to name: String, duration: Double = 1.3, quip: Bool = true) {
        let now = CACurrentMediaTime()
        startMorph(shape(name), duration)
        if name == "blob" {
            mode = .resting
            nextChange = now + Double.random(in: restSeconds)
            nextWobble = now + duration + 1
        } else {
            mode = .showing
            nextChange = now + Double.random(in: holdSeconds)
            if quip, Double.random(in: 0...1) < 0.65, let q = quipFor(name) { say(q) }
            if name == "poop" { flyUntil = now + nextChange - now + 1 }
        }
    }

    private func quipFor(_ shape: String) -> String? {
        let r = Double.random(in: 0...1)
        switch attitude {
        case .sweet: return idleLines[.sweet]!.randomElement()
        case .snarky: return quips[shape]?.randomElement()
        case .mean: return r < 0.5 ? idleLines[.mean]!.randomElement() : quips[shape]?.randomElement()
        case .unhinged:
            if let p = profaneQuips[shape], r < 0.55 { return p.randomElement() }
            return r < 0.8 ? idleLines[.unhinged]!.randomElement() : quips[shape]?.randomElement()
        }
    }

    private func wobble() {
        startMorph(shape("blob"), 2.2)
        nextWobble = CACurrentMediaTime() + Double.random(in: 2.5...4)
    }

    // MARK: Tricks
    private var mouthScreen: CGPoint {
        let er = min(0.16, eyeR * 0.38)
        let m = toView(P(eyeC.x, eyeC.y - er * 1.6))
        let o = window?.frame.origin ?? .zero
        return P(o.x + m.x, o.y + m.y)
    }

    func spit(count: Int = 1, talk: Bool = true) {
        squishUntil = CACurrentMediaTime() + 0.5
        if talk { say(pick(spitLines), seconds: 3) }
        sfx("Bottle")
        for k in 0..<count {
            let m = NSEvent.mouseLocation
            overlay.spit(from: mouthScreen,
                         to: P(m.x + CGFloat.random(in: -18...18), m.y + CGFloat.random(in: -18...18)),
                         delay: Double(k) * 0.15)
        }
    }

    func blep() {
        blepUntil = CACurrentMediaTime() + 2.5
        if Double.random(in: 0...1) < 0.5 { say(blepLines.randomElement()!, seconds: 2) }
    }

    func flyVisit() {
        flyUntil = CACurrentMediaTime() + 14
        if Double.random(in: 0...1) < 0.5 { say(flyLines.randomElement()!, seconds: 2.5) }
    }

    func dodge(from mouse: CGPoint) {
        guard let w = window, let screen = w.screen ?? NSScreen.main else { return }
        let c = P(w.frame.minX + center.x, w.frame.minY + center.y)
        var dx = c.x - mouse.x, dy = c.y - mouse.y
        let len = max(1, hypot(dx, dy)); dx /= len; dy /= len
        let vf = screen.visibleFrame
        var f = w.frame
        f.origin.x += dx * 140; f.origin.y += dy * 90
        if f.minX < vf.minX || f.maxX > vf.maxX { f.origin.x = w.frame.origin.x - dx * 140 }  // cornered: go the other way
        f.origin.x = min(max(f.origin.x, vf.minX), vf.maxX - f.width)
        f.origin.y = min(max(f.origin.y, vf.minY), vf.maxY - f.height)
        NSAnimationContext.runAnimationGroup { ctx in
            ctx.duration = 0.22
            w.animator().setFrame(f, display: true)
        }
        UserDefaults.standard.set(NSStringFromPoint(f.origin), forKey: "windowOrigin")
        say(pick(dodgeLines), seconds: 2)
    }

    private func rage() {
        let now = CACurrentMediaTime()
        angryUntil = now + 3.5
        shakeUntil = now + 1.2
        pokeTimes = []
        sfx("Basso")
        say(pick(rageLines), seconds: 3.5)
        DispatchQueue.main.asyncAfter(deadline: .now() + 0.6) { [weak self] in self?.spit(count: 4, talk: false) }
    }

    private func randomTrick() {
        var options = ["blep", "fly"]
        if attitude == .snarky { options.append("spit") }
        if attitude.rawValue >= Attitude.mean.rawValue { options += ["spit", "spit"] }
        switch options.randomElement()! {
        case "spit": spit(count: Int.random(in: 1...3))
        case "fly": flyVisit()
        default: blep()
        }
    }

    // MARK: Scheduling
    private func schedule(_ now: CFTimeInterval) {
        if now > nextBlink { blinkUntil = now + 0.13; nextBlink = now + Double.random(in: 2...6) }

        if now > nextSleepCheck {
            nextSleepCheck = now + 5
            let hour = Calendar.current.component(.hour, from: Date())
            let night = hour >= 4 && hour < 11   // bedtime: 4am–11am
            if !sleeping && night && now - lastInteraction > 120 && mode == .resting {
                sleeping = true
                morph(to: "blob", duration: 2)
            } else if sleeping && !night {
                sleeping = false
            }
        }
        if sleeping { return }

        switch mode {
        case .resting:
            if now > nextChange { morph(to: randomShapeName()) }
            else if now > nextWobble { wobble() }
        case .showing:
            if now > nextChange { morph(to: "blob") }
        }
        if now > nextTrick {
            nextTrick = now + Double.random(in: trickSeconds)
            randomTrick()
        }
        if hungry && now > nextNag {
            say(pick(hungryLines))
            nextNag = now + Double.random(in: 90...200)
        }
        if now > nextChatter {
            nextChatter = now + 60 * Double.random(in: chatterMinutes)
            if Gemini.key != nil {
                askGemini("(Nobody said anything. Make a short unprompted remark about life on this desktop or about the human.)")
            }
        }
        // Mean blobs don't like being approached.
        if attitude.rawValue >= Attitude.mean.rawValue, dragStart == nil, let w = window {
            let m = NSEvent.mouseLocation
            let near = hypot(m.x - (w.frame.minX + center.x), m.y - (w.frame.minY + center.y)) < 70
            if near && !mouseWasNear && now > nextDodge && Double.random(in: 0...1) < 0.45 {
                nextDodge = now + 25
                dodge(from: m)
            }
            mouseWasNear = near
        }
    }

    // MARK: Frame
    private func tick() {
        let now = CACurrentMediaTime()
        schedule(now)
        let t = CGFloat(min(1, (now - morphStart) / morphDur))
        morphT = t
        let e = easeOutBack(t)
        base = (0..<N).map { lerpP(from.pts[$0], to.pts[$0], e) }
        eyeC = lerpP(from.eye, to.eye, e)
        eyeR = lerp(from.eyeR, to.eyeR, min(1, e))

        let ph = CGFloat(now)
        let breathe: CGFloat = sleeping ? 0.04 * sin(ph * 1.2) : 0.025 * sin(ph * 2)
        var sx: CGFloat = 1 + breathe, sy: CGFloat = 1 - breathe
        if now < squishUntil {
            let k = CGFloat((squishUntil - now) / 0.5)
            let a = 0.2 * k * cos((1 - k) * .pi * 3)
            sx *= 1 + a; sy *= 1 - a
        }
        if dragged && dragStart != nil { sx *= 0.92; sy *= 1.1 }
        let k = 1 - 0.12 * sin(.pi * t)
        sxy = P(sx * k, sy * k)
        shakeX = now < shakeUntil ? 5 * sin(ph * 70) : 0

        rendered = base.enumerated().map { i, p in
            let fi = CGFloat(i)
            let w = 1 + 0.012 * sin(ph * 4 + fi * 0.35) + 0.008 * sin(ph * 2.3 - fi * 0.6)
            return toView(P(p.x * w, p.y * w))
        }

        flyVisible = now < flyUntil || (to.name == "poop" && t > 0.6)
        if flyVisible {
            let a = toView(P(0, 0.8))
            flyPos = P(min(max(a.x + 60 * sin(ph * 2.3) + 10 * sin(ph * 13), 10), bounds.width - 10),
                       min(max(a.y + 25 * sin(ph * 3.1) + 8 * cos(ph * 17), 10), bounds.height - 10))
        }

        if !bubble.isHidden && now > bubbleHideAt { bubble.isHidden = true }
        needsDisplay = true
    }

    /// Unit space → view space, squashing from the bottom so it stays "sitting".
    private func toView(_ p: CGPoint) -> CGPoint {
        P(center.x + shakeX + p.x * sxy.x * R, center.y - R + (p.y + 1) * sxy.y * R)
    }

    private func decoAlpha(_ name: String) -> CGFloat {
        to.name == name ? morphT : (from.name == name ? 1 - morphT : 0)
    }

    // MARK: Drawing
    override func draw(_ dirtyRect: NSRect) {
        guard rendered.count > 2 else { return }
        let now = CACurrentMediaTime()
        let ph = CGFloat(now)

        let xs = rendered.map(\.x), minY = rendered.map(\.y).min()!
        let sw = (xs.max()! - xs.min()!) * 0.8
        NSColor(white: 0, alpha: 0.18).setFill()
        NSBezierPath(ovalIn: NSRect(x: bounds.midX - sw / 2, y: minY - 6, width: sw, height: 10)).fill()

        let path = NSBezierPath()
        path.move(to: rendered[0])
        rendered.dropFirst().forEach { path.line(to: $0) }
        path.close()
        NSColor.black.setFill()
        path.fill()

        // Dumpster fire
        let fire = decoAlpha("dumpster")
        if fire > 0.01 {
            let lid = toView(shape("dumpster").norm(P(0, 0.3)))
            let flick = 0.8 + 0.2 * sin(ph * 23)
            NSGraphicsContext.saveGraphicsState()
            path.addClip()
            NSGradient(starting: NSColor(red: 1, green: 0.3, blue: 0, alpha: fire * flick),
                       ending: NSColor(red: 1, green: 0.9, blue: 0.2, alpha: fire * flick))?
                .draw(in: NSRect(x: 0, y: lid.y, width: bounds.width, height: bounds.height - lid.y), angle: 90)
            NSGraphicsContext.restoreGraphicsState()
        }

        NSGradient(colors: [NSColor(white: 1, alpha: 0.25), NSColor(white: 1, alpha: 0), NSColor(white: 1, alpha: 0)],
                   atLocations: [0, 0.35, 1], colorSpace: .deviceRGB)?
            .draw(in: path, relativeCenterPosition: NSPoint(x: -0.4, y: 0.55))

        // Tombstone engraving
        let rip = decoAlpha("tombstone")
        if rip > 0.01 {
            let c = toView(shape("tombstone").norm(P(0, 0.15)))
            let col = NSColor(white: 1, alpha: rip)
            drawCentered("R.I.P.", at: P(c.x, c.y + 18), size: 20, weight: .black, color: col)
            drawCentered("your", at: P(c.x, c.y - 4), size: 11, weight: .semibold, color: col)
            drawCentered("productivity", at: P(c.x, c.y - 18), size: 11, weight: .semibold, color: col)
        }

        drawFace(now)

        if flyVisible { drawFly(ph) }
        if sleeping { drawZzz(ph) }
    }

    private func drawFace(_ now: CFTimeInterval) {
        var alpha = max(0, min(1, (eyeR - 0.12) / 0.08))
        alpha *= 1 - decoAlpha("tombstone")
        guard alpha > 0.02 else { return }
        let c = toView(eyeC)
        let er = min(0.16, eyeR * 0.38) * R * sxy.x
        let happy = now < happyUntil && !sleeping
        let angry = now < angryUntil && !sleeping
        let open: CGFloat = (sleeping || now < blinkUntil) ? 0.08 : (hungry ? 0.5 : 1)
        let wf = window?.frame ?? .zero
        let m = NSEvent.mouseLocation
        let look = flyVisible ? flyPos : P(m.x - wf.minX, m.y - wf.minY)
        let white = NSColor(white: 1, alpha: alpha)

        for side: CGFloat in [-1, 1] {
            let ec = P(c.x + side * er * 1.3, c.y)
            if happy {
                let arc = NSBezierPath()
                arc.appendArc(withCenter: P(ec.x, ec.y - er * 0.4), radius: er * 0.8, startAngle: 25, endAngle: 155)
                arc.lineWidth = er * 0.45
                arc.lineCapStyle = .round
                white.setStroke()
                arc.stroke()
                continue
            }
            let h = er * 2 * open
            let eye = NSBezierPath(ovalIn: NSRect(x: ec.x - er, y: ec.y - h / 2, width: er * 2, height: h))
            white.setFill()
            eye.fill()
            NSGraphicsContext.saveGraphicsState()
            eye.addClip()
            var dx = look.x - ec.x, dy = look.y - ec.y
            let len = max(1, hypot(dx, dy)); dx /= len; dy /= len
            let reach = er * 0.45 * min(1, len / 80)
            let pr = er * (angry ? 0.35 : 0.55)
            NSColor.black.setFill()
            NSBezierPath(ovalIn: NSRect(x: ec.x + dx * reach - pr, y: ec.y + dy * reach - pr,
                                        width: pr * 2, height: pr * 2)).fill()
            NSGraphicsContext.restoreGraphicsState()

            if angry {   // furious little eyebrows
                let brow = NSBezierPath()
                brow.move(to: P(ec.x - side * er * 1.0, ec.y + er * 1.5))
                brow.line(to: P(ec.x + side * er * 0.9, ec.y + er * 0.9))
                brow.lineWidth = er * 0.4
                brow.lineCapStyle = .round
                white.setStroke()
                brow.stroke()
            }
        }

        if now < blepUntil && !sleeping {   // tongue
            let mouth = P(c.x, c.y - er * 1.4)
            let wag = er * 0.15 * sin(CGFloat(now) * 9)
            let tongue = NSBezierPath(roundedRect: NSRect(x: mouth.x - er * 0.55 + wag, y: mouth.y - er * 1.5,
                                                          width: er * 1.1, height: er * 1.5),
                                      xRadius: er * 0.55, yRadius: er * 0.55)
            NSColor(red: 1, green: 0.45, blue: 0.6, alpha: alpha).setFill()
            tongue.fill()
            let crease = NSBezierPath()
            crease.move(to: P(mouth.x + wag, mouth.y - er * 0.2))
            crease.line(to: P(mouth.x + wag, mouth.y - er * 0.9))
            crease.lineWidth = 1.5
            NSColor(red: 0.8, green: 0.25, blue: 0.4, alpha: alpha).setStroke()
            crease.stroke()
        }
    }

    private func drawFly(_ ph: CGFloat) {
        let flap = 3 + 2 * sin(ph * 80)
        let wingCol = NSColor(white: 1, alpha: 0.75)
        for side: CGFloat in [-1, 1] {
            let wing = NSBezierPath(ovalIn: NSRect(x: flyPos.x + side * 3 - 3, y: flyPos.y + 1, width: 6, height: flap + 2))
            wingCol.setFill(); wing.fill()
            NSColor.black.setStroke(); wing.lineWidth = 0.8; wing.stroke()
        }
        NSColor.black.setFill()
        NSBezierPath(ovalIn: NSRect(x: flyPos.x - 4, y: flyPos.y - 3, width: 8, height: 6)).fill()
    }

    private func drawZzz(_ ph: CGFloat) {
        let top = toView(P(0.45, 0.9))
        for k in 0..<3 {
            let phase = (ph * 0.45 + CGFloat(k) / 3).truncatingRemainder(dividingBy: 1)
            drawCentered("z", at: P(top.x + 10 + phase * 30, top.y + phase * 55), size: 10 + phase * 12,
                         weight: .heavy, color: NSColor(white: 0, alpha: 1 - phase))
        }
    }

    // MARK: Speech bubble
    func say(_ text: String, seconds: Double = 4) {
        label.stringValue = text
        let maxW: CGFloat = 196
        let s = label.cell!.cellSize(forBounds: NSRect(x: 0, y: 0, width: maxW, height: 400))
        let w = min(maxW, ceil(s.width)), h = ceil(s.height)
        let bw = w + 20, bh = h + 14
        bubble.frame = NSRect(x: (bounds.width - bw) / 2, y: bounds.height - bh - 4, width: bw, height: bh)
        label.frame = NSRect(x: 10, y: 7, width: w, height: h)
        bubble.isHidden = false
        bubbleHideAt = CACurrentMediaTime() + seconds
    }

    // MARK: Mouse
    override func mouseDown(with event: NSEvent) {
        dragStart = NSEvent.mouseLocation
        winStart = window?.frame.origin ?? .zero
        dragged = false
    }
    override func mouseDragged(with event: NSEvent) {
        guard let s = dragStart else { return }
        let m = NSEvent.mouseLocation
        let dx = m.x - s.x, dy = m.y - s.y
        if abs(dx) + abs(dy) > 3 { dragged = true }
        window?.setFrameOrigin(NSPoint(x: winStart.x + dx, y: winStart.y + dy))
    }
    override func mouseUp(with event: NSEvent) {
        lastInteraction = CACurrentMediaTime()
        if dragged, let o = window?.frame.origin {
            UserDefaults.standard.set(NSStringFromPoint(o), forKey: "windowOrigin")
        } else {
            poke()
        }
        dragStart = nil
        dragged = false
    }

    private func poke() {
        let now = CACurrentMediaTime()
        squishUntil = now + 0.5
        sfx("Pop")
        if sleeping {
            sleeping = false
            angryUntil = now + 2
            say(pick(wakeLines), seconds: 3)
            return
        }
        pokeTimes = pokeTimes.filter { now - $0 < 3 } + [now]
        if pokeTimes.count >= 5 { rage(); return }
        if attitude.rawValue >= Attitude.mean.rawValue { angryUntil = now + 1 } else { happyUntil = now + 0.8 }
        say(pick(pokeLines), seconds: 2.5)
        if Double.random(in: 0...1) < 0.3 { morph(to: randomShapeName(), quip: false) }
    }

    // MARK: Menu
    override func menu(for event: NSEvent) -> NSMenu? {
        lastInteraction = CACurrentMediaTime()
        let m = NSMenu()
        let filled = Int(hunger / 10)
        let bar = String(repeating: "▮", count: filled) + String(repeating: "▯", count: 10 - filled)
        m.addItem(NSMenuItem(title: "Hunger  \(bar)", action: nil, keyEquivalent: ""))
        m.addItem(.separator())
        add(m, "Feed some ink", #selector(feed))
        add(m, "Morph now!", #selector(morphNow))
        let sub = NSMenu()
        for n in ["blob"] + shapeNames {
            let it = NSMenuItem(title: n.capitalized, action: #selector(become(_:)), keyEquivalent: "")
            it.target = self
            it.representedObject = n
            sub.addItem(it)
        }
        let subItem = NSMenuItem(title: "Become…", action: nil, keyEquivalent: "")
        subItem.submenu = sub
        m.addItem(subItem)
        add(m, "Spit at me", #selector(spitAtMe))
        add(m, "Stick your tongue out", #selector(blepNow))
        m.addItem(.separator())

        let att = NSMenu()
        for a in Attitude.allCases {
            let it = NSMenuItem(title: a.title, action: #selector(setAttitude(_:)), keyEquivalent: "")
            it.target = self
            it.tag = a.rawValue
            it.state = a == attitude ? .on : .off
            att.addItem(it)
        }
        let attItem = NSMenuItem(title: "Attitude", action: nil, keyEquivalent: "")
        attItem.submenu = att
        m.addItem(attItem)
        let snd = NSMenuItem(title: "Sounds", action: #selector(toggleSounds), keyEquivalent: "")
        snd.target = self
        snd.state = (UserDefaults.standard.object(forKey: "sounds") as? Bool ?? true) ? .on : .off
        m.addItem(snd)
        m.addItem(.separator())
        add(m, Gemini.key == nil ? "Talk (needs a Gemini key)…" : "Talk to \(petName)…", #selector(talk))
        add(m, "Gemini API key…", #selector(setKey))
        m.addItem(.separator())
        add(m, "Quit \(petName)", #selector(quit))
        return m
    }
    private func add(_ m: NSMenu, _ title: String, _ sel: Selector) {
        let it = NSMenuItem(title: title, action: sel, keyEquivalent: "")
        it.target = self
        m.addItem(it)
    }

    @objc private func feed() {
        sleeping = false
        lastFed = Date()
        happyUntil = CACurrentMediaTime() + 2.5
        morph(to: "heart", quip: false)
        sfx("Purr")
        say(pick(feedLines))
    }
    @objc private func morphNow() { sleeping = false; morph(to: randomShapeName()) }
    @objc private func become(_ sender: NSMenuItem) {
        sleeping = false
        if let n = sender.representedObject as? String { morph(to: n) }
    }
    @objc private func spitAtMe() { sleeping = false; spit(count: Int.random(in: 2...4)) }
    @objc private func blepNow() { sleeping = false; blep() }
    @objc private func setAttitude(_ sender: NSMenuItem) {
        Attitude.current = Attitude(rawValue: sender.tag) ?? .snarky
        let hello: [Attitude: String] = [.sweet: "aww, hi bestie! 💖", .snarky: "back to my normal charming self.",
                                         .mean: "oh, you want honesty? fine.", .unhinged: "oh, it's fucking ON now."]
        say(hello[Attitude.current]!)
    }
    @objc private func toggleSounds() {
        let on = UserDefaults.standard.object(forKey: "sounds") as? Bool ?? true
        UserDefaults.standard.set(!on, forKey: "sounds")
    }
    @objc private func quit() { NSApp.terminate(nil) }

    @objc private func setKey() {
        guard let k = prompt("Gemini API key",
                             "Paste a free key from aistudio.google.com (leave empty to remove it). It's saved in this app's preferences on your Mac.",
                             secure: true) else { return }
        let key = k.trimmingCharacters(in: .whitespacesAndNewlines)
        UserDefaults.standard.set(key, forKey: "geminiKey")
        say(key.isEmpty ? "brain removed. blorp." : "ooh, a brain. 🧠")
    }

    @objc private func talk() {
        sleeping = false
        if Gemini.key == nil { setKey(); if Gemini.key == nil { return } }
        guard let text = prompt("Talk to \(petName)", "Say something to your blob:"), !text.isEmpty else { return }
        say("…", seconds: 20)
        askGemini(text)
    }

    private func askGemini(_ text: String) {
        let mood = hungry ? "starving and cranky" : "well-fed and smug"
        Gemini.ask(text, mood: mood) { [weak self] reply, shape, action in
            guard let self else { return }
            if let reply { self.say(reply, seconds: 6) }
            if let shape, shape == "blob" || shapeNames.contains(shape) { self.morph(to: shape, quip: false) }
            switch action {
            case "spit": self.spit(count: 2, talk: false)
            case "blep": self.blep()
            case "fly": self.flyVisit()
            case "dodge": self.dodge(from: NSEvent.mouseLocation)
            default: break
            }
        }
    }
}

// MARK: - App
final class AppDelegate: NSObject, NSApplicationDelegate {
    var window: NSWindow!
    var overlay: SplatOverlay!

    func applicationDidFinishLaunching(_ notification: Notification) {
        // Edit menu so ⌘V works when pasting your API key (the menu bar itself stays hidden).
        let mainMenu = NSMenu()
        let editItem = NSMenuItem()
        let edit = NSMenu(title: "Edit")
        edit.addItem(withTitle: "Cut", action: #selector(NSText.cut(_:)), keyEquivalent: "x")
        edit.addItem(withTitle: "Copy", action: #selector(NSText.copy(_:)), keyEquivalent: "c")
        edit.addItem(withTitle: "Paste", action: #selector(NSText.paste(_:)), keyEquivalent: "v")
        edit.addItem(withTitle: "Select All", action: #selector(NSText.selectAll(_:)), keyEquivalent: "a")
        editItem.submenu = edit
        mainMenu.addItem(editItem)
        NSApp.mainMenu = mainMenu

        overlay = SplatOverlay()

        let size = NSSize(width: 240, height: 300)
        let vf = NSScreen.main?.visibleFrame ?? NSRect(x: 0, y: 0, width: 1440, height: 900)
        var origin = NSPoint(x: vf.maxX - size.width - 40, y: vf.minY + 20)
        if let s = UserDefaults.standard.string(forKey: "windowOrigin") {
            let p = NSPointFromString(s)
            if NSScreen.screens.contains(where: { $0.frame.contains(p) }) { origin = p }
        }
        window = NSWindow(contentRect: NSRect(origin: origin, size: size),
                          styleMask: [.borderless], backing: .buffered, defer: false)
        window.isOpaque = false
        window.backgroundColor = .clear
        window.hasShadow = false
        window.level = .floating
        window.collectionBehavior = [.canJoinAllSpaces, .stationary, .ignoresCycle]
        window.contentView = BlobView(frame: NSRect(origin: .zero, size: size), overlay: overlay)
        window.orderFrontRegardless()
    }
}

let app = NSApplication.shared
let delegate = AppDelegate()
app.delegate = delegate
app.setActivationPolicy(.accessory)   // no Dock icon — it just lives on your desktop
app.run()
