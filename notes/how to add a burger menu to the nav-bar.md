Yes — this is the classic checkbox hack: a hidden <input type="checkbox"> + a <label> styled as the hamburger icon. The checkbox's :checked state (toggled by clicking the label) drives the CSS that shows/hides the menu — zero JS needed.

HTML changes
html
<header>
    <nav>
        <div class="nav-brand">
            <img id="nav-logo" src="../images/homePage/logo.png" alt="Logo">
        </div>

        <input type="checkbox" id="nav-toggle" class="nav-toggle">
        <label for="nav-toggle" class="nav-burger">
            <span></span>
            <span></span>
            <span></span>
        </label>

        <div id="nav-links-header">
            <a href="../index.html">Home</a>
            <a href="pricing.html">Pricing</a>
            <a href="careers.html">Careers</a>
            <a href="about.html">About</a>
        </div>
    </nav>
</header>

The checkbox and label are the toggle mechanism; they sit between .nav-brand and #nav-links-header in the markup so the CSS sibling selector (~) can reach both.

CSS additions
css
/* Hide the checkbox itself — it's just state, not visible UI */
.nav-toggle {
    display: none;
}

/* Hide the burger icon on desktop */
.nav-burger {
    display: none;
    flex-direction: column;
    justify-content: space-between;
    width: 28px;
    height: 20px;
    cursor: pointer;
    z-index: 20;
}

.nav-burger span {
    display: block;
    height: 3px;
    width: 100%;
    background: var(--header-link-text);
    border-radius: 2px;
    transition: transform 0.3s ease, opacity 0.3s ease;
}

@media (max-width: 768px) {
    nav {
        position: relative;
        height: 80px;
    }

    #nav-logo {
        width: 100px;
        height: 100px;
    }

    /* Show the burger, hide the full-size link row */
    .nav-burger {
        display: flex;
    }

    #nav-links-header {
        position: absolute;
        top: 80px;
        left: 0;
        right: 0;
        flex-direction: column;
        align-items: center;
        gap: 0;
        background-color: var(--hero-footer-background);
        max-height: 0;
        overflow: hidden;
        transition: max-height 0.3s ease;
        z-index: 10;
    }

    #nav-links-header a {
        padding: 16px 0;
        width: 100%;
        text-align: center;
    }

    /* The actual toggle: checked checkbox reveals the menu */
    .nav-toggle:checked ~ #nav-links-header {
        max-height: 300px;
    }

    /* Animate burger into an X when open */
    .nav-toggle:checked ~ .nav-burger span:nth-child(1) {
        transform: translateY(8.5px) rotate(45deg);
    }
    .nav-toggle:checked ~ .nav-burger span:nth-child(2) {
        opacity: 0;
    }
    .nav-toggle:checked ~ .nav-burger span:nth-child(3) {
        transform: translateY(-8.5px) rotate(-45deg);
    }
}
Why this works
The checkbox is the state. :checked is a native CSS pseudo-class — no JS needed to track "is the menu open."
The <label for="nav-toggle"> is the click target. Clicking any label toggles its associated checkbox, so the burger icon (a styled label) opens/closes the menu just by being clicked.
~ (general sibling combinator) lets .nav-toggle:checked style any later sibling — that's how #nav-links-header and .nav-burger's spans react to the checkbox state despite not being nested inside it. This is why the checkbox, burger, and links all need to be siblings in the HTML — the hack breaks if they're nested differently.
max-height transition instead of display — you can't transition display: none → block, but max-height: 0 → 300px with overflow: hidden gives you a smooth slide-down animation for free.
At exactly 360px: 100px logo + burger icon fit comfortably in one row (justify-content: space-between still applies), and the full link list only appears in the collapsible dropdown below — so there's no overflow math to fight this time.

One accessibility note: this pattern isn't keyboard/screen-reader ideal (no aria-expanded state, and some screen readers announce checkboxes oddly in a nav context) — worth keeping in mind if this needs to meet accessibility requirements later, since a JS-driven aria-expanded toggle on a real <button> is the more robust long-term solution.