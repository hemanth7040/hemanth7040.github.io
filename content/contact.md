---
title: "Contact"
description: "Get in touch for hiring, collaborations, or technical discussions."
layout: "simple"
showAuthor: false
showDate: false
showPagination: false
---

Whether you are looking to hire, collaborate on a project, or discuss DevOps and MLOps topics — I would love to hear from you. I typically respond within one to two business days.

<div class="contact-wrapper">
  <div class="contact-links">
    <a href="mailto:hemanthkumarmotukuri@gmail.com" class="contact-link">
      📧 hemanthkumarmotukuri@gmail.com
    </a>
    <a href="https://linkedin.com/in/YOUR-HANDLE" class="contact-link" target="_blank">
      💼 LinkedIn Profile
    </a>
  </div>
</div>

---

<form action="https://formspree.io/f/xzdybnae" method="POST" class="contact-form">

  <input type="hidden" name="_subject" value="New message from portfolio contact form">
  <input type="hidden" name="_next" value="https://YOUR-SITE.com/thanks">
  <input type="text" name="_gotcha" style="display:none">

  <div class="form-group">
    <label for="name">Full Name</label>
    <input type="text" name="name" id="name" required placeholder="Jane Smith">
  </div>

  <div class="form-group">
    <label for="email">Email Address</label>
    <input type="email" name="_replyto" id="email" required placeholder="jane@company.com">
  </div>

  <div class="form-group">
    <label>Subject</label>
    <div class="dd-wrap">
      <div class="dd-selected" id="ddSelected">
        <span id="ddLabel">Select a topic</span>
        <svg class="dd-arrow" viewBox="0 0 16 16" fill="none" stroke="#94a3b8" stroke-width="2">
          <polyline points="4 6 8 10 12 6"/>
        </svg>
      </div>
      <div class="dd-list" id="ddList">
        <div class="dd-item" data-value="hiring">Hiring / Job Opportunity</div>
        <div class="dd-item" data-value="collaboration">Project Collaboration</div>
        <div class="dd-item" data-value="consulting">Consulting Inquiry</div>
        <div class="dd-item" data-value="general">General Question</div>
      </div>
      <input type="hidden" name="subject" id="ddHidden">
    </div>
  </div>

  <div class="form-group">
    <label for="message">Message</label>
    <textarea name="message" id="message" rows="6" required placeholder="Describe your project, opportunity, or question in a few lines."></textarea>
  </div>

  <button type="submit" class="submit-btn">Send Message →</button>

</form>

<small>Your information is used solely to respond to your inquiry and is never shared with third parties.</small>

<style>
.dd-wrap {
  position: relative;
}

.dd-selected {
  width: 100%;
  background: #1e2a3a;
  border: 1px solid #2d3f55;
  border-radius: 8px;
  padding: 10px 14px;
  font-size: 14px;
  color: #94a3b8;
  cursor: pointer;
  display: flex;
  justify-content: space-between;
  align-items: center;
  transition: border-color .15s;
  user-select: none;
}

.dd-selected.open {
  border-color: #4a90d9;
  border-radius: 8px 8px 0 0;
}

.dd-selected.has-value {
  color: #e2e8f0;
}

.dd-arrow {
  width: 14px;
  height: 14px;
  transition: transform .2s;
  flex-shrink: 0;
}

.dd-selected.open .dd-arrow {
  transform: rotate(180deg);
}

.dd-list {
  display: none;
  position: absolute;
  top: 100%;
  left: 0;
  right: 0;
  background: #1e2a3a;
  border: 1px solid #4a90d9;
  border-top: none;
  border-radius: 0 0 8px 8px;
  z-index: 99;
  overflow: hidden;
}

.dd-list.open {
  display: block;
}

.dd-item {
  padding: 10px 14px;
  font-size: 14px;
  color: #94a3b8;
  cursor: pointer;
  transition: background .12s, color .12s;
}

.dd-item:hover {
  background: #253447;
  color: #e2e8f0;
}

.dd-item.active {
  color: #4a90d9;
  background: #1a2d42;
}

.submit-btn {
  background: #2563eb;
  color: #ffffff;
  border: none;
  padding: 11px 28px;
  border-radius: 8px;
  font-size: 14px;
  font-weight: 500;
  cursor: pointer;
  transition: background .15s, transform .1s;
  letter-spacing: .01em;
}

.submit-btn:hover {
  background: #1d4ed8;
}

.submit-btn:active {
  transform: scale(0.98);
}
</style>

<script>
(function () {
  const sel    = document.getElementById('ddSelected');
  const list   = document.getElementById('ddList');
  const lbl    = document.getElementById('ddLabel');
  const hidden = document.getElementById('ddHidden');

  sel.addEventListener('click', function () {
    sel.classList.toggle('open');
    list.classList.toggle('open');
  });

  list.querySelectorAll('.dd-item').forEach(function (item) {
    item.addEventListener('click', function () {
      list.querySelectorAll('.dd-item').forEach(function (i) {
        i.classList.remove('active');
      });
      item.classList.add('active');
      lbl.textContent  = item.textContent;
      hidden.value     = item.dataset.value;
      sel.classList.add('has-value');
      sel.classList.remove('open');
      list.classList.remove('open');
    });
  });

  document.addEventListener('click', function (e) {
    if (!sel.contains(e.target) && !list.contains(e.target)) {
      sel.classList.remove('open');
      list.classList.remove('open');
    }
  });
})();
</script>
