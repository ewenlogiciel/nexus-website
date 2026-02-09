<script setup>
import { onMounted } from 'vue'

const baseUrl = import.meta.env.BASE_URL

const roles = [
  "Gameplay Programming (C#)",
  "VR Mechanics & Physics",
  "Level Design & Lighting",
  "UI Integration"
]

const assets = [
  { name: "Pistolet Laser", type: "Modèle 3D", img: baseUrl + "gun.png" },
  { name: "Baril futuriste", type: "Modèle 3D", img: baseUrl + "barrel.png" },
  { name: "Baril futuriste v2", type: "Modèle 3D", img: baseUrl + "barrel2.png" }
]

const stack = [
  "Unity 2022.3.62f1",
  "C# / MonoBehaviour",
  "Meta XR Interaction SDK",
  "Git / GitHub"
]

const treeOutput = `nexus/
├── Builds/
├── Library/
├── Packages/
├── Assets/
│   ├── Oculus Hands/
│   ├── Prefabs/
│   ├── Scenes/
│   ├── Scripts/
│   ├── Materials/
│   ├── sounds/
│   ├── XR/
│   └── gun/
└── ProjectSettings/`

const codeSnippet = `// CibleGlitch.cs
using UnityEngine;
using System.Collections;

public class CibleGlitch : MonoBehaviour
{
    [Header("Apparition")]
    [SerializeField] private float dureeApparition = 0.5f;
    [SerializeField] private AnimationCurve courbeApparition = AnimationCurve.EaseInOut(0, 0, 1, 1);

    [Header("Glitch")]
    [SerializeField] private bool glitchActif = true;
    [SerializeField] private float intervalleGlitchMin = 2f;
    [SerializeField] private float intervalleGlitchMax = 5f;
    [SerializeField] private float dureeGlitch = 0.2f;
    [SerializeField] private float intensiteGlitch = 0.5f;

    [Header("Disparition")]
    [SerializeField] private float dureeVie = 5f;
    [SerializeField] private float dureeDisparition = 0.5f;

    private MeshRenderer meshRenderer;
    private Material materielInstance;
    private Vector3 positionInitiale;
    private Vector3 scaleInitiale;
    private Color couleurInitiale;
    private float intensiteEmissionInitiale;

    void Start()
    {
        meshRenderer = GetComponent<MeshRenderer>();
        if (meshRenderer != null)
        {
            materielInstance = meshRenderer.material;
            couleurInitiale = materielInstance.GetColor("_BaseColor");
            intensiteEmissionInitiale = materielInstance.GetColor("_EmissionColor").r;
        }

        positionInitiale = transform.localPosition;
        scaleInitiale = transform.localScale;

        // Commence invisible
        transform.localScale = Vector3.zero;

        // Lance la séquence
        StartCoroutine(SequenceCible());
    }

    IEnumerator SequenceCible()
    {
        // Apparition
        yield return StartCoroutine(Apparaitre());

        // Vie de la cible avec glitchs
        float tempsEcoule = 0f;
        float prochainGlitch = Random.Range(intervalleGlitchMin, intervalleGlitchMax);

        while (tempsEcoule < dureeVie)
        {
            if (glitchActif && tempsEcoule >= prochainGlitch)
            {
                StartCoroutine(EffetGlitch());
                prochainGlitch = tempsEcoule + Random.Range(intervalleGlitchMin, intervalleGlitchMax);
            }

            tempsEcoule += Time.deltaTime;
            yield return null;
        }

        // Disparition
        yield return StartCoroutine(Disparaitre());

        // Détruit l'objet
        Destroy(gameObject);
    }

    IEnumerator Apparaitre()
    {
        float tempsEcoule = 0f;

        while (tempsEcoule < dureeApparition)
        {
            float t = courbeApparition.Evaluate(tempsEcoule / dureeApparition);

            // Scale progressif
            transform.localScale = scaleInitiale * t;

            // Effet de glitch sur l'apparition
            if (Random.value > 0.7f)
            {
                Vector3 offset = Random.insideUnitSphere * intensiteGlitch * (1 - t);
                transform.localPosition = positionInitiale + offset;
            }
            else
            {
                transform.localPosition = positionInitiale;
            }

            tempsEcoule += Time.deltaTime;
            yield return null;
        }

        transform.localScale = scaleInitiale;
        transform.localPosition = positionInitiale;
    }

    IEnumerator Disparaitre()
    {
        float tempsEcoule = 0f;

        while (tempsEcoule < dureeDisparition)
        {
            float t = 1 - (tempsEcoule / dureeDisparition);

            // Scale décroissant
            transform.localScale = scaleInitiale * t;

            // Effet de glitch sur la disparition
            if (Random.value > 0.5f)
            {
                Vector3 offset = Random.insideUnitSphere * intensiteGlitch * (1 - t);
                transform.localPosition = positionInitiale + offset;
            }

            // Flicker de l'émission
            if (meshRenderer != null && Random.value > 0.5f)
            {
                float emission = intensiteEmissionInitiale * t * Random.Range(0.5f, 1.5f);
                Color emissionColor = couleurInitiale * emission;
                materielInstance.SetColor("_EmissionColor", emissionColor);
            }

            tempsEcoule += Time.deltaTime;
            yield return null;
        }
    }

    IEnumerator EffetGlitch()
    {
        float tempsEcoule = 0f;

        while (tempsEcoule < dureeGlitch)
        {
            // Glitch de position
            if (Random.value > 0.5f)
            {
                Vector3 offset = Random.insideUnitSphere * intensiteGlitch;
                transform.localPosition = positionInitiale + offset;
            }
            else
            {
                transform.localPosition = positionInitiale;
            }

            // Glitch de scale
            if (Random.value > 0.7f)
            {
                float scaleMultiplier = Random.Range(0.8f, 1.2f);
                transform.localScale = scaleInitiale * scaleMultiplier;
            }

            // Flicker d'émission
            if (meshRenderer != null && Random.value > 0.6f)
            {
                float emission = intensiteEmissionInitiale * Random.Range(0.3f, 2f);
                Color emissionColor = couleurInitiale * emission;
                materielInstance.SetColor("_EmissionColor", emissionColor);
            }

            tempsEcoule += Time.deltaTime;
            yield return new WaitForSeconds(0.05f);
        }

        // Retour à la normale
        transform.localPosition = positionInitiale;
        transform.localScale = scaleInitiale;
        if (meshRenderer != null)
        {
            Color emissionColor = couleurInitiale * intensiteEmissionInitiale;
            materielInstance.SetColor("_EmissionColor", emissionColor);
        }
    }

    void OnDestroy()
    {
        // Nettoie l'instance du matériau
        if (materielInstance != null)
        {
            Destroy(materielInstance);
        }
    }
}
`

onMounted(() => {
  const observer = new IntersectionObserver((entries) => {
    entries.forEach(entry => {
      if (entry.isIntersecting) {
        entry.target.classList.add('in-view')
        observer.unobserve(entry.target)
      }
    })
  }, { threshold: 0.08, rootMargin: '0px 0px -40px 0px' })

  document.querySelectorAll('[data-reveal]').forEach(el => {
    observer.observe(el)
  })
})
</script>

<template>
  <div class="min-h-screen">

    <!-- ═══════════════════ NAV ═══════════════════ -->
    <nav class="fixed top-0 w-full z-50 bg-[#0a0a0a]/90 backdrop-blur-lg border-b border-border">
      <div class="max-w-[1200px] mx-auto px-6 h-14 flex items-center justify-between">
        <span class="font-display font-bold text-sm tracking-[0.3em] uppercase text-white">
          Nexus
        </span>
        <a
          :href="baseUrl + 'wr507d_vr_ewen_davanzo.apk'"
          download="wr507d_vr_ewen_davanzo.apk"
          class="bg-accent text-black text-[11px] font-bold tracking-wider uppercase px-5 py-2.5 hover:bg-white transition-colors duration-300"
        >
          Télécharger l'APK
        </a>
      </div>
    </nav>

    <!-- ═══════════════════ HERO ═══════════════════ -->
    <section class="h-screen flex flex-col items-center justify-center text-center px-6 relative">
      <span
        class="font-mono text-[11px] tracking-[0.4em] uppercase text-accent mb-8"
        data-reveal="fade"
      >
        VR Experience
      </span>

      <h1
        class="font-display font-extrabold text-[clamp(4.5rem,15vw,13rem)] leading-[0.85] tracking-tighter uppercase text-white"
        data-reveal="up"
      >
        NEXUS
      </h1>

      <div
        class="w-16 h-[1px] bg-accent mt-10 mb-10"
        data-reveal="fade"
        style="--delay: 0.2s"
      ></div>

      <p
        class="text-lg md:text-xl text-[#888] max-w-lg leading-relaxed font-light"
        data-reveal="up"
        style="--delay: 0.3s"
      >
        60 secondes. Un pistolet laser.<br>
        <span class="text-[#4ade80]">Visez le vert.</span>
        <span class="text-[#f87171]">Évitez le rouge.</span>
      </p>

      <!-- Scroll indicator -->
      <div class="absolute bottom-10 flex flex-col items-center gap-3">
        <span class="font-mono text-[9px] tracking-[0.3em] text-[#333] uppercase">Scroll</span>
        <div class="w-px h-10 scroll-line"></div>
      </div>
    </section>

    <!-- ═══════════════════ 01 — GAMEPLAY ═══════════════════ -->
    <section class="border-t border-border">
      <div class="max-w-[1200px] mx-auto px-6 py-24 md:py-32">
        <div class="flex items-baseline gap-4 mb-14" data-reveal="up">
          <span class="font-mono text-accent text-sm">01</span>
          <h2 class="font-display font-bold text-3xl md:text-4xl text-white">Le Gameplay</h2>
        </div>

        <div class="grid md:grid-cols-2 gap-12 md:gap-16 items-center">
          <div data-reveal="up" style="--delay: 0.1s">
            <p class="text-[#999] leading-relaxed text-base md:text-lg">
              Nexus est un jeu de tir rythmé en VR. Une partie dure exactement
              <strong class="text-white font-semibold">1 minute</strong>.
              Le joueur scanne son environnement, récupère un pistolet laser et tire
              sur un maximum de
              <span class="text-[#4ade80]">cibles vertes</span>
              qui apparaissent avec des effets de glitch.
            </p>
            <p class="text-[#999] leading-relaxed mt-5 text-base md:text-lg">
              Attention : toucher une
              <span class="text-[#f87171]">cible rouge</span>
              fait perdre des points.
            </p>
          </div>

          <div
            class="aspect-[4/3] bg-surface border border-border overflow-hidden"
            data-reveal="up"
            style="--delay: 0.2s"
          >
            <video
              controls
              playsinline
              class="w-full h-full object-contain"
              :src="baseUrl + 'gameplay.mp4'"
            >
              Votre navigateur ne supporte pas la vidéo.
            </video>
          </div>
        </div>
      </div>
    </section>

    <!-- ═══════════════════ 02 — ASSETS ═══════════════════ -->
    <section class="border-t border-border bg-surface/40">
      <div class="max-w-[1200px] mx-auto px-6 py-24 md:py-32">
        <div class="flex items-baseline gap-4 mb-14" data-reveal="up">
          <span class="font-mono text-accent text-sm">02</span>
          <h2 class="font-display font-bold text-3xl md:text-4xl text-white">Assets</h2>
        </div>

        <div class="grid md:grid-cols-3 gap-4">
          <div
            v-for="(asset, i) in assets"
            :key="i"
            class="group border border-border bg-[#0a0a0a] hover:border-accent/30 transition-all duration-500"
            data-reveal="up"
            :style="`--delay: ${0.1 + i * 0.1}s`"
          >
            <div class="aspect-[4/3] overflow-hidden bg-[#0d0d0d]">
              <img
                :src="asset.img"
                :alt="asset.name"
                class="w-full h-full object-cover opacity-50 group-hover:opacity-90 group-hover:scale-[1.03] transition-all duration-700 ease-out"
              >
            </div>
            <div class="p-4 flex justify-between items-center border-t border-border">
              <span class="text-sm font-medium text-white">{{ asset.name }}</span>
              <span class="font-mono text-[10px] text-[#444]">{{ asset.type }}</span>
            </div>
          </div>
        </div>
      </div>
    </section>

    <!-- ═══════════════════ 03 — DÉVELOPPEMENT ═══════════════════ -->
    <section class="border-t border-border">
      <div class="max-w-[1200px] mx-auto px-6 py-24 md:py-32">
        <div class="flex items-baseline gap-4 mb-14" data-reveal="up">
          <span class="font-mono text-accent text-sm">03</span>
          <h2 class="font-display font-bold text-3xl md:text-4xl text-white">Développement</h2>
        </div>

        <div class="grid lg:grid-cols-3 gap-6">
          <!-- Unity screenshot -->
          <div class="lg:col-span-2" data-reveal="up" style="--delay: 0.1s">
            <div class="flex justify-between items-center mb-3">
              <span class="text-sm font-medium text-white">Interface Unity</span>
              <span class="font-mono text-[10px] text-[#333] tracking-wider uppercase">Scene View / Inspector</span>
            </div>
            <div class="border border-border overflow-hidden bg-surface">
              <img
                :src="baseUrl + 'unity-interface.png'"
                alt="Interface Unity"
                class="w-full h-auto opacity-70 hover:opacity-100 transition-opacity duration-700"
              >
            </div>
            <p class="text-[11px] text-[#444] mt-3 font-mono">
              Vue de la scène principale avec les gizmos de navigation et les zones de spawn.
            </p>
          </div>

          <!-- File tree -->
          <div class="lg:col-span-1" data-reveal="up" style="--delay: 0.2s">
            <div class="flex justify-between items-center mb-3">
              <span class="text-sm font-medium text-white">Arborescence</span>
              <span class="font-mono text-[10px] text-[#333] tracking-wider uppercase">File System</span>
            </div>
            <div class="border border-border bg-[#0c0c0c] p-5 min-h-[300px] lg:h-[calc(100%-2rem)]">
              <div class="flex gap-1.5 mb-5 pb-3 border-b border-border">
                <div class="w-2.5 h-2.5 rounded-full bg-[#ff5f57]"></div>
                <div class="w-2.5 h-2.5 rounded-full bg-[#febc2e]"></div>
                <div class="w-2.5 h-2.5 rounded-full bg-[#28c840]"></div>
              </div>
              <pre class="font-mono text-xs text-[#666] leading-relaxed whitespace-pre select-all">{{ treeOutput }}</pre>
            </div>
          </div>
        </div>
      </div>
    </section>

    <!-- ═══════════════════ 04 — CODE ═══════════════════ -->
    <section class="border-t border-border bg-surface/40">
      <div class="max-w-[1000px] mx-auto px-6 py-24 md:py-32">
        <div class="flex items-baseline gap-4 mb-14" data-reveal="up">
          <span class="font-mono text-accent text-sm">04</span>
          <h2 class="font-display font-bold text-3xl md:text-4xl text-white">Code</h2>
        </div>

        <div class="border border-border overflow-hidden" data-reveal="up" style="--delay: 0.1s">
          <!-- Editor title bar -->
          <div class="flex items-center justify-between px-4 py-3 bg-[#0f0f0f] border-b border-border">
            <div class="flex items-center gap-3">
              <div class="flex gap-1.5">
                <div class="w-2.5 h-2.5 rounded-full bg-[#ff5f57]"></div>
                <div class="w-2.5 h-2.5 rounded-full bg-[#febc2e]"></div>
                <div class="w-2.5 h-2.5 rounded-full bg-[#28c840]"></div>
              </div>
              <span class="font-mono text-xs text-[#555]">CibleGlitch.cs</span>
            </div>
            <span class="font-mono text-[10px] text-[#2a2a2a] tracking-wider uppercase">C#</span>
          </div>

          <!-- Code content -->
          <pre class="p-6 overflow-x-auto overflow-y-auto max-h-[520px] text-[13px] font-mono text-[#999] leading-relaxed bg-[#0a0a0a]"><code>{{ codeSnippet }}</code></pre>
        </div>
      </div>
    </section>

    <!-- ═══════════════════ 05 — DÉVELOPPEUR & STACK ═══════════════════ -->
    <section class="border-t border-border">
      <div class="max-w-[1200px] mx-auto px-6 py-24 md:py-32">
        <div class="grid md:grid-cols-2 gap-16">

          <!-- Developer -->
          <div data-reveal="up">
            <div class="flex items-baseline gap-4 mb-10">
              <span class="font-mono text-accent text-sm">05</span>
              <h2 class="font-display font-bold text-3xl text-white">Le Développeur</h2>
            </div>

            <div class="border border-border p-6 hover:border-accent/20 transition-colors duration-500">
              <div class="flex items-center gap-4 mb-6">
                <div class="w-11 h-11 bg-accent flex items-center justify-center text-black font-display font-bold text-lg">
                  E
                </div>
                <div>
                  <h3 class="text-lg font-display font-bold text-white leading-tight">Ewen D'Avanzo</h3>
                  <span class="font-mono text-[10px] tracking-[0.15em] uppercase text-accent">Solo Developer</span>
                </div>
              </div>

              <p class="text-sm text-[#666] mb-6 leading-relaxed">
                Projet réalisé intégralement en autonomie. Responsable de l'ensemble du pipeline de production :
              </p>

              <ul class="space-y-3">
                <li
                  v-for="role in roles"
                  :key="role"
                  class="flex items-center gap-3 text-sm text-[#bbb]"
                >
                  <span class="w-1 h-1 bg-accent flex-shrink-0"></span>
                  {{ role }}
                </li>
              </ul>
            </div>
          </div>

          <!-- Stack -->
          <div data-reveal="up" style="--delay: 0.15s">
            <div class="flex items-baseline gap-4 mb-10">
              <span class="font-mono text-accent text-sm invisible">05</span>
              <h2 class="font-display font-bold text-3xl text-white">Stack Technique</h2>
            </div>

            <div class="space-y-3">
              <div
                v-for="(tech, i) in stack"
                :key="i"
                class="flex items-center gap-4 p-4 border border-border hover:border-accent/20 transition-colors duration-500"
              >
                <span class="font-mono text-xs text-accent w-6">{{ String(i + 1).padStart(2, '0') }}</span>
                <span class="text-sm text-[#bbb]">{{ tech }}</span>
              </div>
            </div>
          </div>

        </div>
      </div>
    </section>

    <!-- ═══════════════════ FOOTER / DOWNLOAD ═══════════════════ -->
    <footer class="border-t border-border">
      <div class="max-w-[1200px] mx-auto px-6 py-24 md:py-32 text-center">
        <span
          class="font-mono text-[11px] text-accent tracking-[0.3em] uppercase block mb-6"
          data-reveal="fade"
        >
          Meta Quest 2 / 3
        </span>

        <h2
          class="font-display font-bold text-4xl md:text-5xl text-white mb-12"
          data-reveal="up"
          style="--delay: 0.1s"
        >
          Prêt à tester Nexus ?
        </h2>

        <a
          :href="baseUrl + 'wr507d_vr_ewen_davanzo.apk'"
          download="wr507d_vr_ewen_davanzo.apk"
          class="inline-block bg-accent text-black font-display font-bold text-base tracking-wide uppercase px-10 py-4 hover:bg-white transition-colors duration-300"
          data-reveal="up"
          style="--delay: 0.2s"
        >
          Télécharger le projet
        </a>

        <p
          class="font-mono text-[11px] text-[#333] mt-8"
          data-reveal="fade"
          style="--delay: 0.3s"
        >
          Version 1.0.0
        </p>
      </div>

      <div class="border-t border-border py-6 text-center">
        <p class="font-mono text-[11px] text-[#2a2a2a]">
          © 2026 Ewen D'Avanzo
        </p>
      </div>
    </footer>

  </div>
</template>

<style scoped>
.scroll-line {
  background: linear-gradient(to bottom, var(--color-accent, #c8ff00) 0%, transparent 100%);
  animation: scroll-pulse 2.5s cubic-bezier(0.4, 0, 0.6, 1) infinite;
}

@keyframes scroll-pulse {
  0%, 100% { opacity: 0.15; }
  50% { opacity: 0.6; }
}
</style>
