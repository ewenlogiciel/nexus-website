<script setup>
import { ref } from 'vue'

const baseUrl = import.meta.env.BASE_URL

const myRoles = [
  "Gameplay Programming (C#)",
  "VR Mechanics & Physics",
  "Level Design & Lighting",
  "UI Integration"
]

const projectTree = [
  {
    name: "nexus",
    type: "root",
    children: [
      { name: "Builds", type: "folder" },
      { name: "Library", type: "folder" },
      { name: "Packages", type: "folder" },
      {
        name: "Assets",
        type: "folder-open",
        children: [
          { name: "Oculus Hands", type: "folder" },
          { name: "Prefabs", type: "folder" },
          { name: "Scenes", type: "folder" },
          { name: "Scripts", type: "folder" },
          { name: "Materials", type: "folder" },
          { name: "sounds", type: "folder" },
          { name: "XR", type: "folder" },
          { name: "gun", type: "folder" }
        ]
      },
      { name: "ProjectSettings", type: "folder" }
    ]
  }
]

const mainAssets = [
  {
    name: "Pistolet Laser",
    role: "Modèle 3d",
    img: baseUrl + "gun.png"
  },
  {
    name: "Baril futuriste",
    role: "Modèle 3d",
    img: baseUrl + "barrel.png"
  },
  {
    name: "Baril futuriste",
    role: "Modèle 3d",
    img: baseUrl + "barrel2.png"
  }
]

const codeSnippet = `
// CibleGlitch.cs
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
            materielInstance = meshRenderer.material; // Crée une instance du matériau
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
            yield return new WaitForSeconds(0.05f); // 20 FPS pour l'effet glitch
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
</script>

<template>
  <div class="min-h-screen bg-black selection:bg-cyan-500 selection:text-black">

    <nav class="fixed top-0 w-full z-50 border-b border-white/10 bg-black/80 backdrop-blur-md">
      <div class="max-w-7xl mx-auto px-6 h-16 flex items-center justify-between">
        <span class="text-2xl font-bold tracking-widest neon-text text-white">nexus</span>
        <a
            :href="baseUrl + 'wr507d_vr_ewen_davanzo.apk'"
            download="wr507d_vr_ewen_davanzo.apk"
            class="px-4 py-2 bg-white text-black font-bold rounded hover:bg-cyan-400 transition-colors duration-300"
        >
          Télécharger l'APK
        </a>
      </div>
    </nav>

    <section class="relative h-screen flex flex-col items-center justify-center text-center px-4 overflow-hidden">
      <div class="absolute inset-0 bg-[radial-gradient(circle_at_center,_var(--tw-gradient-stops))] from-purple-900/20 via-black to-black opacity-50"></div>

      <h1 class="text-7xl md:text-9xl font-black mb-6 tracking-tighter bg-clip-text text-transparent bg-gradient-to-r from-cyan-400 to-purple-600 animate-pulse">
        nexus
      </h1>
      <p class="text-xl md:text-2xl text-gray-400 max-w-2xl z-10">
        60 secondes. Un pistolet laser.
        <br>Visez le vert. Évitez le rouge.
      </p>
    </section>

    <section class="max-w-6xl mx-auto px-6 py-20 grid md:grid-cols-2 gap-16 items-center">
      <div>
        <h2 class="text-3xl font-bold mb-6 text-cyan-400">Le Gameplay</h2>
        <p class="text-gray-300 leading-relaxed mb-6">
          Nexus est un jeu de tir rythmé en VR. Une partie dure exactement <strong>1 minute</strong>.
          Le joueur doit scanner son environnement, récupérer un pistolet laser et tirer sur un maximum de <span class="text-green-400">cibles vertes</span> qui apparaissent avec des effets de glitch.
          <br><br>
          Attention : toucher une <span class="text-red-500">cible rouge</span> fait perdre des points.
        </p>
      </div>

      <div class="aspect-[4/3] bg-black rounded-xl border border-white/10 overflow-hidden shadow-[0_0_30px_rgba(0,240,255,0.1)]">

        <video
            controls
            playsinline
            class="w-full h-full object-contain"
            :src="baseUrl + 'gameplay.mp4'"
        >
          Votre navigateur ne supporte pas la vidéo.
        </video>

      </div>
    </section>

    <section class="bg-gray-900/30 py-20 border-y border-white/5">
      <div class="max-w-7xl mx-auto px-6">
        <h2 class="text-3xl font-bold mb-10 text-center text-white">Assets utilisés</h2>

        <div class="grid md:grid-cols-3 gap-6">
          <div v-for="(asset, index) in mainAssets" :key="index" class="col-span-1 border border-white/10 rounded-lg p-2 bg-black/50 hover:border-cyan-400/50 transition-colors duration-300">
            <h3 class="text-sm font-mono text-gray-400 mb-2 border-b border-white/10 pb-2 flex justify-between">
              <span>{{ asset.name }}</span>
            </h3>
            <div class="aspect-video bg-gray-800 rounded flex items-center justify-center overflow-hidden relative group">
              <img :src="asset.img" class="w-full h-full object-cover opacity-60 group-hover:opacity-100 transition-opacity duration-300" alt="Asset Preview">
            </div>
            <p class="text-xs text-cyan-400 mt-2 text-right">{{ asset.role }}</p>
          </div>
        </div>

      </div>
    </section>

    <section class="max-w-7xl mx-auto px-6 py-20 border-t border-white/5">
        <h2 class="text-3xl font-bold mb-10 text-center text-white">Environnement de Développement</h2>

        <div class="grid lg:grid-cols-3 gap-8">

          <div class="lg:col-span-2 space-y-4">
            <div class="flex justify-between items-end">
              <h3 class="text-xl font-bold text-cyan-400">Interface Unity</h3>
              <span class="text-xs text-gray-500 font-mono">SCENE VIEW / INSPECTOR</span>
            </div>

            <div class="bg-gray-900 rounded-lg border border-white/10 p-1 overflow-hidden shadow-2xl">
              <img
                  :src="baseUrl + 'unity-interface.png'"
                  alt="Interface Unity"
                  class="w-full h-auto rounded opacity-80 hover:opacity-100 transition-opacity duration-500"
              >
            </div>
            <p class="text-sm text-gray-400 italic">
              Vue de la scène principale avec les gizmos de navigation et les zones de spawn.
            </p>
          </div>

          <div class="lg:col-span-1">
            <div class="flex justify-between items-end mb-4">
              <h3 class="text-xl font-bold text-purple-400">Arborescence</h3>
              <span class="text-xs text-gray-500 font-mono">FILE SYSTEM</span>
            </div>

            <div class="bg-[#0c0c0c] rounded-lg border border-white/10 p-6 font-mono text-sm h-full shadow-[inset_0_0_20px_rgba(0,0,0,0.8)] overflow-y-auto max-h-[436px]">
              <div class="flex gap-2 mb-4 border-b border-white/5 pb-2">
                <div class="w-2 h-2 rounded-full bg-red-500"></div>
                <div class="w-2 h-2 rounded-full bg-yellow-500"></div>
                <div class="w-2 h-2 rounded-full bg-green-500"></div>
              </div>

              <div class="space-y-1">
                <div v-for="root in projectTree" :key="root.name">
                  <div class="text-gray-300 font-bold mb-2">📁 {{ root.name }}</div>

                  <div v-for="child in root.children" :key="child.name" class="ml-4 border-l border-white/10 pl-4">
                    <div :class="child.type === 'folder-open' ? 'text-white' : 'text-gray-500'">
                      {{ child.type === 'folder-open' ? '📂' : '📁' }} {{ child.name }}
                    </div>

                    <div v-if="child.children" class="ml-4 mt-1 space-y-1 border-l border-cyan-500/20 pl-4">
                      <div v-for="sub in child.children" :key="sub.name" class="text-cyan-300/80 hover:text-cyan-300 hover:bg-white/5 cursor-default transition-colors rounded px-1">
                        ↳ 📁 {{ sub.name }}
                      </div>
                      <div class="text-gray-600 italic text-xs mt-2 ml-2">... et autres ressources</div>
                    </div>
                  </div>

                </div>
              </div>

            </div>
          </div>

        </div>
      </section>

    <section class="max-w-5xl mx-auto px-6 py-20">
      <h2 class="text-3xl font-bold mb-6 text-purple-500">Exemple de code (C#)</h2>
      <div class="bg-[#1e1e1e] rounded-xl overflow-hidden shadow-2xl border border-white/10">
        <div class="flex items-center px-4 py-2 bg-[#252526] border-b border-black">
          <div class="flex gap-2">
            <div class="w-3 h-3 rounded-full bg-red-500"></div>
            <div class="w-3 h-3 rounded-full bg-yellow-500"></div>
            <div class="w-3 h-3 rounded-full bg-green-500"></div>
          </div>
          <span class="ml-4 text-xs text-gray-400 font-mono">Target.cs</span>
        </div>
        <pre class="p-6 overflow-x-auto text-sm font-mono text-gray-300"><code>{{ codeSnippet }}</code></pre>
      </div>
    </section>

    <section class="max-w-7xl mx-auto px-6 py-20 grid md:grid-cols-2 gap-12">

      <div>
        <h3 class="text-2xl font-bold mb-6 border-l-4 border-cyan-400 pl-4 text-white">Le Développeur</h3>

        <div class="bg-white/5 p-6 rounded-lg border border-white/10 hover:border-cyan-400/30 transition-colors">
          <div class="flex items-center gap-4 mb-6">
            <div class="w-12 h-12 bg-gradient-to-br from-cyan-400 to-blue-600 rounded-full flex items-center justify-center text-black font-black text-xl">
              E
            </div>
            <div>
              <h4 class="text-xl font-bold text-white">Ewen D'Avanzo</h4>
              <span class="px-2 py-0.5 rounded text-[10px] uppercase font-bold bg-cyan-400/20 text-cyan-400 border border-cyan-400/30">
                Solo Developer
              </span>
            </div>
          </div>

          <p class="text-sm text-gray-400 mb-4">
            Projet réalisé intégralement en autonomie. Responsable de l'ensemble du pipeline de production :
          </p>

          <ul class="space-y-2">
            <li v-for="role in myRoles" :key="role" class="flex items-center gap-3 text-gray-300 text-sm">
              <span class="w-1.5 h-1.5 bg-cyan-400 rounded-full"></span>
              {{ role }}
            </li>
          </ul>
        </div>
      </div>

      <div>
        <h3 class="text-2xl font-bold mb-6 border-l-4 border-purple-500 pl-4 text-white">Stack Technique</h3>
        <div class="grid grid-cols-1 gap-3">
          <div class="flex items-center gap-3 text-gray-400 bg-black/40 p-3 rounded border border-white/5">
            <span class="w-2 h-2 bg-purple-500 rounded-full"></span>
            Unity 2022.3.62f1
          </div>
          <div class="flex items-center gap-3 text-gray-400 bg-black/40 p-3 rounded border border-white/5">
            <span class="w-2 h-2 bg-purple-500 rounded-full"></span>
            C# / Monobehaviour
          </div>
          <div class="flex items-center gap-3 text-gray-400 bg-black/40 p-3 rounded border border-white/5">
            <span class="w-2 h-2 bg-purple-500 rounded-full"></span>
            Meta XR Interaction SDK
          </div>
          <div class="flex items-center gap-3 text-gray-400 bg-black/40 p-3 rounded border border-white/5">
            <span class="w-2 h-2 bg-purple-500 rounded-full"></span>
            Git / GitHub
          </div>
        </div>
      </div>

    </section>

    <footer id="download" class="bg-gradient-to-t from-cyan-900/20 to-black py-24 text-center border-t border-white/10">
      <h2 class="text-4xl font-bold mb-8 text-white">Prêt à tester Nexus ?</h2>
      <div class="flex flex-col items-center gap-4">

        <a
            :href="baseUrl + 'wr507d_vr_ewen_davanzo.apk'"
            download="wr507d_vr_ewen_davanzo.apk"
            class="group relative px-8 py-4 bg-white text-black font-black text-xl rounded hover:scale-105 transition-transform duration-200 inline-block cursor-pointer"
        >
          <span class="relative z-10">Télécharger le projet</span>
          <div class="absolute inset-0 bg-cyan-400 blur-lg opacity-50 group-hover:opacity-100 transition-opacity"></div>
        </a>

        <p class="text-sm text-gray-500 mt-4">Version 1.0.0 • Compatible Meta Quest 2/3</p>
      </div>
      <div class="mt-20 text-gray-600 text-sm">
        © 2026 Ewen D'Avanzo | Tous droits réservés.
      </div>
    </footer>

  </div>
</template>

<style scoped>
.neon-text {
  text-shadow: 0 0 10px rgba(34, 211, 238, 0.5);
}
</style>