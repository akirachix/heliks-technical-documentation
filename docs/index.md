---
hide:
  - title
---

<div align="center" style="margin-top: 1rem; padding: 0 0.5rem;">

<img src="assets/images/Rica.png" alt="RICA Logo" width="150" style="margin-bottom: 1.5rem;" id="rica-home-logo">

<script>
function updateRicaLogo() {
    const logo = document.getElementById('rica-home-logo');
    if (!logo) return;
    const scheme = document.querySelector('body').getAttribute('data-md-color-scheme') || 'default';
    logo.src = scheme === 'slate' ? 'assets/images/Rica2.png' : 'assets/images/Rica.png';
}
window.addEventListener('load', updateRicaLogo);
const observer = new MutationObserver(updateRicaLogo);
observer.observe(document.body, { attributes: true, attributeFilter: ['data-md-color-scheme'] });
</script>

<div align="center" style="margin: 1rem 0;">
    <img src="assets/images/chain.png" alt="RICA Supply Chain" style="width: 100%; max-width: 1200px; height: auto; border-radius: 12px;">
</div>

<div style="margin-top: 2.5rem; margin-bottom: 3rem;">
    <a href="developer-guide/backend-setup/" style="background: linear-gradient(135deg, #262262, #1a194a); color: white !important; font-weight: 600; padding: 0.8rem 2rem; border-radius: 8px; text-decoration: none !important; margin: 0.5rem; display: inline-block;">Learn More</a>
    <a href="architecture/system-context/" style="background: #0BC08F; color: white !important; font-weight: 600; padding: 0.8rem 2rem; border-radius: 8px; text-decoration: none !important; margin: 0.5rem; display: inline-block;">Get Started</a>
</div>

</div>


## About RICA

RICA provides end-to-end traceability from extraction through processing stages, maintaining an immutable audit trail of all batch movements. The platform enforces strict data validation, role-based access control, and company-level scoping to ensure data integrity and security across multi-tenant operations.

<div style="margin: 1.5rem 0;">
    <a href="https://heliks-informational-website.vercel.app" target="_blank" style="background: #fff; color: #262262 !important; font-weight: 600; padding: 0.7rem 1.8rem; border-radius: 8px; text-decoration: none !important; display: inline-block; border: 2px solid #262262;">More about RICA</a>
</div>

## System Architecture

<div style="display: grid; grid-template-columns: repeat(auto-fit, minmax(250px, 1fr)); gap: 1.5rem; margin: 2rem 0;">

<div class="premium-card">
    <h3 style="margin-top: 0; color: #262262;">Backend Engine</h3>
    <p style="margin-bottom: 0; opacity: 0.85;">FastAPI service with SQLAlchemy ORM, PostgreSQL database, JWT authentication, and automated value calculations using live LME pricing.</p>
</div>

<div class="premium-card">
    <h3 style="margin-top: 0; color: #262262;">Web Dashboard</h3>
    <p style="margin-bottom: 0; opacity: 0.85;">Next.js 16 application with role-based interfaces for operations managers, supply chain managers, and system administrators.</p>
</div>

<div class="premium-card">
    <h3 style="margin-top: 0; color: #262262;">Marketing Portal</h3>
    <p style="margin-bottom: 0; opacity: 0.85;">Informational website showcasing the platform capabilities with optimized single-page navigation.</p>
</div>

</div>

## App Preview

<div style="display: grid; grid-template-columns: repeat(auto-fit, minmax(300px, 1fr)); gap: 1.5rem; margin: 2rem 0;">

<img src="assets/images/Login.png" alt="Login Interface" class="design-mockup">
<img src="assets/images/Activate.png" alt="Activate" class="design-mockup">
<img src="assets/images/batches.png" alt="Batch Management" class="design-mockup">
<img src="assets/images/trace.png" alt="Trace" class="design-mockup">
<img src="assets/images/value.png" alt="Value Calculations" class="design-mockup">
<img src="assets/images/slider.png" alt="Scenario Modelling" class="design-mockup">

</div>

## Project Team

<div align="center" style="margin: 2rem 0;">
    <img src="assets/images/heliks.png" alt="Heliks Logo" width="150">
</div>

<div style="display: flex; flex-wrap: wrap; justify-content: center; gap: 1.5rem; margin: 2.5rem 0;">

<div style="text-align: center; flex: 0 0 160px;">
    <div style="width: 110px; height: 110px; margin: 0 auto 0.8rem auto; border-radius: 50%; overflow: hidden; border: 2px solid #0BC08F; box-shadow: 0 3px 15px rgba(11, 192, 143, 0.25);">
        <img src="assets/images/samantha-linda.jpg" alt="Samantha Linda" style="width: 100%; height: 100%; object-fit: cover;">
    </div>
    <div style="font-size: 0.9rem; font-weight: 600; line-height: 1.3;">Samantha Linda</div>
    <div style="font-size: 0.8rem; margin-top: 0.2rem;">Software Engineer</div>
</div>

<div style="text-align: center; flex: 0 0 160px;">
    <div style="width: 110px; height: 110px; margin: 0 auto 0.8rem auto; border-radius: 50%; overflow: hidden; border: 2px solid #0BC08F; box-shadow: 0 3px 15px rgba(11, 192, 143, 0.25);">
        <img src="assets/images/mailisha-awuor.jpg" alt="Mailisha Awuor" style="width: 100%; height: 100%; object-fit: cover;">
    </div>
    <div style="font-size: 0.9rem; font-weight: 600; line-height: 1.3;">Mailisha Awuor</div>
    <div style="font-size: 0.8rem; margin-top: 0.2rem;">Software Engineer</div>
</div>

<div style="text-align: center; flex: 0 0 160px;">
    <div style="width: 110px; height: 110px; margin: 0 auto 0.8rem auto; border-radius: 50%; overflow: hidden; border: 2px solid #0BC08F; box-shadow: 0 3px 15px rgba(11, 192, 143, 0.25);">
        <img src="assets/images/rose-wanjiru.jpg" alt="Rose Wanjiru" style="width: 100%; height: 100%; object-fit: cover;">
    </div>
    <div style="font-size: 0.9rem; font-weight: 600; line-height: 1.3;">Rose Wanjiru</div>
    <div style="font-size: 0.8rem; margin-top: 0.2rem;">Software Engineer</div>
</div>

<div style="text-align: center; flex: 0 0 160px;">
    <div style="width: 110px; height: 110px; margin: 0 auto 0.8rem auto; border-radius: 50%; overflow: hidden; border: 2px solid #0BC08F; box-shadow: 0 3px 15px rgba(11, 192, 143, 0.25);">
        <img src="assets/images/eyerusalem-gabremedhin.jpg" alt="Eyerusalem Gabremedhin" style="width: 100%; height: 100%; object-fit: cover;">
    </div>
    <div style="font-size: 0.9rem; font-weight: 600; line-height: 1.3;">Eyerusalem Gabremedhin</div>
    <div style="font-size: 0.8rem; margin-top: 0.2rem;">Software Engineer</div>
</div>

<div style="text-align: center; flex: 0 0 160px;">
    <div style="width: 110px; height: 110px; margin: 0 auto 0.8rem auto; border-radius: 50%; overflow: hidden; border: 2px solid #0BC08F; box-shadow: 0 3px 15px rgba(11, 192, 143, 0.25);">
        <img src="assets/images/salvatrice-igabineza.jpg" alt="Salvatrice Igabineza" style="width: 100%; height: 100%; object-fit: cover;">
    </div>
    <div style="font-size: 0.9rem; font-weight: 600; line-height: 1.3;">Salvatrice Igabineza</div>
    <div style="font-size: 0.8rem; margin-top: 0.2rem;">Software Engineer</div>
</div>

</div>

<div align="center" style="margin: 2rem 0;">
    <div style="display: flex; flex-wrap: wrap; justify-content: center; gap: 2rem 3rem;">
        <div style="display: flex; align-items: center; gap: 0.8rem;">
            <div style="width: 40px; height: 40px; border-radius: 50%; background: linear-gradient(135deg, #262262, #1a194a); display: flex; align-items: center; justify-content: center; color: white; font-size: 1.1rem;">✉</div>
            <div style="text-align: left;">
                <div style="font-size: 0.75rem; opacity: 0.7;">Email</div>
                <a href="mailto:heliks@gmail.com" style="font-weight: 600; text-decoration: none; color: inherit;">heliks@gmail.com</a>
            </div>
        </div>
        <div style="display: flex; align-items: center; gap: 0.8rem;">
            <div style="width: 40px; height: 40px; border-radius: 50%; background: linear-gradient(135deg, #0BC08F, #0AA883); display: flex; align-items: center; justify-content: center; color: white; font-size: 1.1rem;">☎</div>
            <div style="text-align: left;">
                <div style="font-size: 0.75rem; opacity: 0.7;">Contact</div>
                <div style="font-weight: 600;">+254789101213</div>
            </div>
        </div>
        <div style="display: flex; align-items: center; gap: 0.8rem;">
            <div style="width: 40px; height: 40px; border-radius: 50%; background: linear-gradient(135deg, #262262, #1a194a); display: flex; align-items: center; justify-content: center;">
                <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="white" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M21 10c0 7-9 13-9 13s-9-6-9-13a9 9 0 0 1 18 0z"></path><circle cx="12" cy="10" r="3"></circle></svg>
            </div>
            <div style="text-align: left;">
                <div style="font-size: 0.75rem; opacity: 0.7;">Location</div>
                <div style="font-weight: 600;">616- Korongo Road</div>
            </div>
        </div>
        <div style="display: flex; align-items: center; gap: 0.8rem;">
            <div style="width: 40px; height: 40px; border-radius: 50%; background: linear-gradient(135deg, #0BC08F, #0AA883); display: flex; align-items: center; justify-content: center;">
                <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="white" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><rect x="2" y="2" width="20" height="20" rx="5" ry="5"></rect><path d="M16 11.37A4 4 0 1 1 12.63 8 4 4 0 0 1 16 11.37z"></path><line x1="17.5" y1="6.5" x2="17.51" y2="6.5"></line></svg>
            </div>
            <div style="text-align: left;">
                <div style="font-size: 0.75rem; opacity: 0.7;">Instagram</div>
                <div style="font-weight: 600;">heliks_developers</div>
            </div>
        </div>
    </div>
</div>
