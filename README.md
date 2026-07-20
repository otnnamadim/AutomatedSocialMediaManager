import os
import pandas as pd
import datetime
from pathlib import Path

class StrategicRoadmapEngine:
    """Dynamically maps the content engine to an external business growth strategy."""
    
    @staticmethod
    def get_current_strategic_slice(brand_key: str, roadmap_csv=Path(__file__).parent / "output" / "brand_strategic_roadmap.csv") -> dict:
        try:
            df = pd.read_csv(roadmap_csv)
            # Find current week starting Saturday/Sunday
            week_start = pd.Timestamp(datetime.date.today()).to_period("W-SAT").start_time
            label = week_start.strftime("%Y-%m-%d") 
            
            # Filter for the specific brand and chronological block
            slice_row = df[(df["Week_Start"] == label) & (df["Account"] == brand_key)]
            
            if not slice_row.empty:
                r = slice_row.iloc[0]
                return {
                    "monthly_theme": r["Monthly_Theme"],
                    "weekly_storyboard": r["Weekly_Storyboard_Arc"],
                    "growth_objective": r["Core_Growth_Objective"]
                }
        except FileNotFoundError:
            print("⚠️ brand_strategic_roadmap.csv not found. Reverting to baseline configurations.")
        
        # Safe Strategic Fallbacks to keep the engine online
        fallbacks = {
            "ozoemena_nnamadim": {
                "monthly_theme": "Rebuilding and Career Pivots",
                "weekly_storyboard": "Documenting data pipelines, pandas optimization, and shifting away from standard accounting toolkits.",
                "growth_objective": "Build professional presence around quantitative economic analysis."
            },
            "aperture_advisory": {
                "monthly_theme": "Technology Integration for Small and Medium Enterprises",
                "weekly_storyboard": "Demonstrating data transparency, AI governance safeguards, and automated infrastructure frameworks.",
                "growth_objective": "Drive inbound discovery for complex consulting engagements."
            }
        }
        return fallbacks.get(brand_key)


# =====================================================================
# SYSTEM PROMPT INJECTOR (REPLACES LOGIC INSIDE STRATEGYENGINE)
# =====================================================================
def generate_weekly_storyboard(brand_key: str, active_pillar: str, performance_data: dict) -> str:
    brand = StrategyEngine.BRAND_CONFIG[brand_key]
    
    # Dynamic Extraction from our brand strategy roadmap
    roadmap = StrategicRoadmapEngine.get_current_strategic_slice(brand_key)
    voice_block = StrategyEngine._format_voice_examples(brand["voice_examples"])
    beats_block = "\n".join(f"- {b}" for b in brand["beats"])

    # Isolate top performing text formats cleanly for the LLM to parse
    top_examples_block = "\n---\n".join(performance_data.get("top_performing_examples", ["No data available"]))

    prompt = f"""
    You are an elite, metrics-driven Social Media Strategist and Editorial Director. Your objective is to synthesize upstream corporate strategy, brand roadmaps, and historical performance analytics to generate high-performing, insightful textual content for X (Twitter).

    ACCOUNT CONTEXT: Generating for channel [{brand_key}]
    BRAND VOICE EXPECTATIONS: {brand['voice']}
    
    {voice_block}

    ## SECTION 1: UPSTREAM STRATEGIC ROADMAP OBJECTIVES
    Your content must align with this predefined structural trajectory. Do not deviate from these parameters:
    - **Current Monthly Strategic Theme:** {roadmap['monthly_theme']}
    - **Current Weekly Storyboard Arc:** {roadmap['weekly_storyboard']}
    - **Core Business Growth Objective:** {roadmap['growth_objective']}
    - **Active Focus Content Pillar:** {active_pillar}

    ## SECTION 2: EMPIRICAL PERFORMANCE LOOP (ANALYTICS FEEDBACK)
    Emulate the structural traits of your top-performing content and explicitly filter out the attributes found in underperforming updates.
    
    - **Top Quartile Format Diagnostics:** {performance_data.get('top_quartile_summary', 'None')}
    - **Bottom Quartile Format Diagnostics (AVOID):** {performance_data.get('bottom_quartile_summary', 'None')}
    
    CRITICAL: Analyze and emulate the syntax patterns, pacing, and hooks of these exact top-performing updates:
    ---
    {top_examples_block}
    ---

    ## SECTION 3: NARRATIVE ARCS & CONSTRAINTS
    - **Insight-Led Framing:** Every post must lead with an immediate, authoritative technical truth, paradox, or hard-hitting data realization. No preambles, introductory hooks, or rhetorical questions.
    - **Structural Formatting:** Use clear line breaks and whitespace. Rely on bold text sparingly for structural emphasis. Use clean markdown bullet points for complex breakdowns.
    - **Image Generation Directive:** BANNED. Do not generate text overlays, alternative text placeholders, or image instructions.

    ## SECTION 4: NARRATIVE BEATS FOR THIS PIPELINE
    Draw from these structural frames across the 5-day cycle:
    {beats_block}

    Output a strict JSON array containing exactly 5 sequential elements mapping directly to this schema:
    {{
      "brand": "{brand['display_name']}",
      "active_pillar": "{active_pillar}",
      "weekly_throughline": "Single sentence summarizing the continuous narrative arc",
      "days": [
        {{
          "day": 1,
          "beat": "Name of narrative beat applied",
          "format": "one of {brand['formats']}",
          "concept": "Detailed description of the analytical angle for this update",
          "needs_verification": []
        }}
      ]
    }}
    """
    return openai_generate_with_fallback(prompt)
