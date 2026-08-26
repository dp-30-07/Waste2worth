# Waste2worth
from flask import Flask, request, render_template_string
import os
import html

from google import genai
from google.genai import types


app = Flask(__name__)


# ============================================================
# GEMINI CLIENT
# ============================================================

client = genai.Client(
    api_key=os.environ.get("GEMINI_API_KEY")
)


# ============================================================
# HOME PAGE
# ============================================================

HTML = """
<!DOCTYPE html>
<html>
<head>

    <title>Waste2Worth</title>

    <meta name="viewport" content="width=device-width, initial-scale=1">

    <style>

        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
            font-family: Arial, sans-serif;
        }

        body {
            background: #f4f8f2;
            color: #18351f;
        }

        .navbar {
            background: white;
            padding: 20px 8%;
            display: flex;
            justify-content: space-between;
            align-items: center;
            box-shadow: 0 2px 10px rgba(0,0,0,0.05);
        }

        .logo {
            font-size: 24px;
            font-weight: bold;
        }

        .logo span {
            color: #3c8c45;
        }

        .tagline {
            font-size: 18px;
        }

        .hero {
            text-align: center;
            padding: 70px 20px 40px;
        }

        .hero h1 {
            font-size: 48px;
            margin-bottom: 15px;
        }

        .hero h1 span {
            color: #3c8c45;
        }

        .hero p {
            font-size: 19px;
            color: #617064;
            max-width: 700px;
            margin: auto;
            line-height: 1.6;
        }

        .upload-box {
            background: white;
            max-width: 650px;
            margin: 40px auto;
            padding: 45px;
            border-radius: 20px;
            box-shadow: 0 8px 30px rgba(0,0,0,0.08);
            text-align: center;
        }

        .upload-icon {
            font-size: 55px;
            margin-bottom: 15px;
        }

        .upload-box h2 {
            margin-bottom: 10px;
        }

        .upload-box p {
            color: #718074;
            margin-bottom: 25px;
        }

        input[type="file"] {
            margin: 15px 0;
            padding: 12px;
            width: 100%;
        }

        button {
            background: #3c8c45;
            color: white;
            border: none;
            padding: 15px 30px;
            border-radius: 10px;
            font-size: 16px;
            cursor: pointer;
        }

        button:hover {
            background: #2f7137;
        }

        button:disabled {
            cursor: not-allowed;
            opacity: 0.7;
        }

        #preview {
            display: none;
            max-width: 300px;
            max-height: 250px;
            margin: 20px auto;
            border-radius: 12px;
        }

        #loadingMessage {
            display: none;
            margin-top: 20px;
            text-align: center;
            color: #3c8c45;
            font-size: 17px;
            font-weight: bold;
        }

        .features {
            display: flex;
            justify-content: center;
            gap: 20px;
            flex-wrap: wrap;
            padding: 30px 8% 50px;
        }

        .card {
            background: white;
            padding: 25px;
            width: 220px;
            border-radius: 15px;
            text-align: center;
            box-shadow: 0 5px 20px rgba(0,0,0,0.05);
        }

        .card .icon {
            font-size: 35px;
            margin-bottom: 10px;
        }

        .calculator {
            background: white;
            max-width: 650px;
            margin: 20px auto 60px;
            padding: 35px;
            border-radius: 20px;
            box-shadow: 0 8px 30px rgba(0,0,0,0.08);
        }

        .calculator h2 {
            text-align: center;
            color: #18351f;
        }

        .calculator-description {
            text-align: center;
            color: #718074;
            margin: 10px 0 25px;
        }

        .calculator label {
            display: block;
            margin-top: 15px;
            font-weight: bold;
        }

        .calculator input {
            width: 100%;
            padding: 12px;
            margin: 8px 0 18px;
            border: 1px solid #ccc;
            border-radius: 8px;
        }

        .calculator button {
            width: 100%;
            margin-top: 5px;
        }

        @media (max-width: 700px) {

            .navbar {
                padding: 18px 5%;
            }

            .tagline {
                display: none;
            }

            .hero h1 {
                font-size: 36px;
            }

            .upload-box {
                margin: 25px 15px;
                padding: 30px 20px;
            }

            .calculator {
                margin: 20px 15px 50px;
            }
        }

    </style>

</head>

<body>


<!-- NAVBAR -->

<div class="navbar">

    <div class="logo">
        🌱 Waste<span>2</span>Worth
    </div>

    <div class="tagline">
        AI for Sustainable Agriculture
    </div>

</div>


<!-- HERO -->

<div class="hero">

    <h1>
        Turn Farm Waste into
        <span>Value</span>
    </h1>

    <p>
        Upload a photo of agricultural waste and discover
        smarter ways to reuse it, reduce waste and create
        new income opportunities.
    </p>

</div>


<!-- UPLOAD SECTION -->

<div class="upload-box">

    <div class="upload-icon">
        📸
    </div>

    <h2>
        Analyze Your Farm Waste
    </h2>

    <p>
        Upload a clear photo of crop residue or agricultural waste.
    </p>


    <form
        action="/analyze"
        method="POST"
        enctype="multipart/form-data"
        id="analyzeForm"
    >

        <input
            type="file"
            name="waste_image"
            accept="image/*"
            id="wasteImage"
            required
        >


        <img
            id="preview"
            alt="Image Preview"
        >


        <br>


        <button
            type="submit"
            id="analyzeButton"
        >
            🔍 Analyze Waste
        </button>


        <div id="loadingMessage">

            ⏳ AI is analyzing your farm waste...

            <br>

            <span style="font-size:14px;">
                Please wait a moment
            </span>

        </div>

    </form>

</div>


<!-- FEATURES -->

<div class="features">

    <div class="card">

        <div class="icon">
            🤖
        </div>

        <h3>
            AI Analysis
        </h3>

        <p>
            Identify agricultural waste.
        </p>

    </div>


    <div class="card">

        <div class="icon">
            ♻️
        </div>

        <h3>
            Reuse Ideas
        </h3>

        <p>
            Find useful ways to reuse waste.
        </p>

    </div>


    <div class="card">

        <div class="icon">
            💰
        </div>

        <h3>
            Value Potential
        </h3>

        <p>
            Explore possible income opportunities.
        </p>

    </div>

</div>


<!-- VALUE CALCULATOR -->

<div class="calculator">

    <h2>
        💰 Waste Value Calculator
    </h2>

    <p class="calculator-description">
        Estimate the potential value of your agricultural waste.
    </p>


    <form
        action="/calculate"
        method="POST"
    >

        <label>
            Waste Quantity (tons)
        </label>

        <input
            type="number"
            name="quantity"
            placeholder="Example: 2"
            min="0"
            step="0.1"
            required
        >


        <label>
            Price per Ton (₹)
        </label>

        <input
            type="number"
            name="price"
            placeholder="Example: 2000"
            min="0"
            required
        >


        <button type="submit">
            💰 Calculate Estimated Value
        </button>

    </form>

</div>


<!-- JAVASCRIPT -->

<script>

    const imageInput =
        document.getElementById("wasteImage");

    const preview =
        document.getElementById("preview");

    const analyzeForm =
        document.getElementById("analyzeForm");

    const analyzeButton =
        document.getElementById("analyzeButton");

    const loadingMessage =
        document.getElementById("loadingMessage");


    imageInput.addEventListener(
        "change",
        function(event) {

            const file =
                event.target.files[0];

            if (file) {

                preview.src =
                    URL.createObjectURL(file);

                preview.style.display =
                    "block";

            }

        }
    );


    analyzeForm.addEventListener(
        "submit",
        function() {

            analyzeButton.disabled =
                true;

            analyzeButton.innerHTML =
                "⏳ Analyzing...";

            loadingMessage.style.display =
                "block";

        }
    );

</script>


</body>
</html>
"""


# ============================================================
# HOME ROUTE
# ============================================================

@app.route("/")
def home():

    return render_template_string(HTML)


# ============================================================
# ANALYZE ROUTE
# ============================================================

@app.route("/analyze", methods=["POST"])
def analyze():

    image = request.files.get("waste_image")


    if not image:

        return """
        <h2>No image uploaded.</h2>
        <a href="/">← Go Back</a>
        """


    try:

        image_bytes = image.read()


        # ====================================================
        # GEMINI PROMPT
        # ====================================================

        prompt = """
You are Waste2Worth AI, an agricultural waste analysis assistant.

Carefully inspect the uploaded image before answering.

IMPORTANT:

- Do NOT automatically say the image is not agricultural waste.
- Look carefully for crop residues, harvested crops, discarded vegetables,
  fruits, leaves, stalks, husks, straw, hay, coconut waste, sugarcane waste,
  rice straw, corn stover, animal-related agricultural residues,
  or other farm waste.
- If agricultural waste is clearly visible, identify it.
- If the image shows a farm, field, harvested crop, crop residue,
  or discarded agricultural material, consider that as agricultural context.
- Only say "This does not appear to be agricultural waste."
  when there is genuinely no visible agricultural waste.

If agricultural waste IS visible, respond using exactly this structure:

Waste2Worth AI Analysis

1. Waste Type
   Identify the likely agricultural waste material and briefly explain what is visible.

2. Suggested Reuse
   Give 2 or 3 practical ways this waste can be reused.

3. Value Opportunity
   Explain how the farmer could potentially create income or save money from this waste.

4. Sustainability Impact
   Explain how reusing this waste can reduce pollution, burning,
   landfill waste, or resource usage.

5. Possible Buyers or Users
   Mention realistic types of people, businesses, or industries
   that could use this material.

6. Best Reuse Option
   Based on the visible waste and the practical reuse methods suggested above,
   recommend the single most suitable reuse option and briefly explain why.

7. Estimated Value Opportunity
   If the image provides enough visual evidence, give a rough potential
   value range for the visible waste and clearly label it as an estimate.

   Do not claim an exact quantity, weight, price, or income when it cannot
   be determined from the image.

   If there is not enough information, say:

   "A reliable value estimate requires the waste quantity and local market price."

Keep the answer simple, clear, and suitable for a student hackathon demonstration.

Do not invent details that cannot reasonably be seen in the image.
"""


        # ====================================================
        # TRY GEMINI
        # ====================================================

        try:

            response = client.models.generate_content(

                model="gemini-3.5-flash",

                contents=[
                    prompt,

                    types.Part.from_bytes(
                        data=image_bytes,
                        mime_type=image.mimetype
                    )
                ]
            )


            result = response.text

            ai_status = "🤖 Gemini AI Analysis"


        # ====================================================
        # FALLBACK IF GEMINI ERROR / QUOTA
        # ====================================================

        except Exception as ai_error:

            print(
                "Gemini unavailable:",
                ai_error
            )


            result = """
Waste2Worth AI Analysis

1. Waste Type
   Agricultural Crop Residue

2. Suggested Reuse

   • Animal Feed and Bedding:
   Crop residues such as stalks and leaves can be collected,
   chopped and used as suitable livestock bedding or roughage.

   • Biomass Energy:
   Dry agricultural residues can be converted into biomass
   briquettes or pellets for energy production.

   • Composting and Mulching:
   Crop residues can be returned to the soil as organic mulch
   or compost material.

3. Value Opportunity

   Farmers can potentially sell collected agricultural residues
   to livestock farms, compost producers or biomass-energy businesses.

   Reusing suitable residue on the farm can also reduce
   fertilizer and waste-management costs.

4. Sustainability Impact

   Reusing agricultural residues can reduce open-field burning,
   air pollution and unnecessary waste.

   Returning suitable residues to the soil can also improve
   soil organic matter.

5. Possible Buyers or Users

   • Livestock and dairy farms
   • Compost producers
   • Biomass energy companies
   • Local farmers
   • Nurseries

6. Best Reuse Option

   Conservation Mulching

   Returning suitable crop residue to the field is a practical
   option because it requires less transportation and can help
   protect soil moisture and reduce erosion.

7. Estimated Value Opportunity

   A reliable value estimate requires the waste quantity
   and local market price.

Note:
Gemini AI is temporarily unavailable, so this is a demonstration analysis.
"""

            ai_status = "🧪 Demo Analysis Mode"


        # ====================================================
        # CLEAN RESULT
        # ====================================================

        result = result.replace(
            "###",
            ""
        )

        result = result.replace(
            "**",
            ""
        )

        result = result.replace(
            "*",
            "•"
        )


        safe_result = html.escape(
            result
        )


        # ====================================================
        # ANALYSIS RESULT PAGE
        # ====================================================

        return f"""
<!DOCTYPE html>
<html>

<head>

    <title>Waste2Worth Analysis</title>

    <meta name="viewport"
          content="width=device-width, initial-scale=1">

    <style>

        * {{
            box-sizing: border-box;
            font-family: Arial, sans-serif;
        }}

        body {{
            margin: 0;
            background: #f4f8f2;
            color: #18351f;
            padding: 40px 20px;
        }}

        .page {{
            max-width: 850px;
            margin: auto;
        }}

        .header {{
            text-align: center;
            margin-bottom: 30px;
        }}

        .header h1 {{
            font-size: 38px;
            margin-bottom: 10px;
        }}

        .header p {{
            color: #718074;
            font-size: 16px;
        }}

        .result-card {{
            background: white;
            padding: 40px;
            border-radius: 20px;
            border-left: 6px solid #3c8c45;
            box-shadow: 0 8px 30px rgba(0,0,0,0.08);
        }}

        .status {{
            text-align: center;
            background: #f1f8f2;
            padding: 12px;
            border-radius: 10px;
            margin: 20px 0 30px;
            color: #3c8c45;
            font-weight: bold;
        }}

        .result {{
            white-space: pre-wrap;
            font-size: 17px;
            line-height: 1.8;
            color: #26382a;
        }}

        .buttons {{
            text-align: center;
            margin-top: 35px;
        }}

        .button {{
            display: inline-block;
            text-decoration: none;
            color: white;
            background: #3c8c45;
            padding: 14px 25px;
            border-radius: 10px;
            margin: 5px;
            font-size: 16px;
        }}

        .print-button {{
            background: #18351f;
            border: none;
            cursor: pointer;
        }}

        @media print {{

            .buttons,
            .status {{
                display: none;
            }}

            body {{
                background: white;
            }}

            .result-card {{
                box-shadow: none;
                border: none;
            }}

        }}

    </style>

</head>


<body>

<div class="page">


    <div class="header">

        <h1>
            🌱 Waste2Worth AI
        </h1>

        <p>
            Smart Agricultural Waste Analysis
        </p>

    </div>


    <div class="result-card">

        <h2 style="text-align:center;">
            🔍 Analysis Result
        </h2>


        <div class="status">

            {ai_status}

        </div>


        <div class="result">{safe_result}</div>


        <div class="buttons">

            <button
                class="button print-button"
                onclick="window.print()"
            >
                📄 Download / Save Report
            </button>


            <a
                href="/"
                class="button"
            >
                ← Analyze Another Image
            </a>

        </div>

    </div>

</div>

</body>
</html>
"""


    # ========================================================
    # OUTER ERROR HANDLER
    # ========================================================

    except Exception as e:

        print(
            "Application Error:",
            e
        )


        return f"""
<!DOCTYPE html>
<html>

<head>

    <title>Analysis Error</title>

</head>

<body style="
    font-family:Arial;
    text-align:center;
    padding:80px;
    background:#f4f8f2;
    color:#18351f;
">

    <h1>
        ⚠️ Analysis Error
    </h1>

    <p style="
        color:#617064;
        font-size:17px;
        line-height:1.6;
        max-width:700px;
        margin:25px auto;
    ">

        Something went wrong while processing the image.

    </p>

    <a href="/">
        ← Go Back
    </a>

</body>

</html>
"""


# ============================================================
# VALUE CALCULATOR
# ============================================================

@app.route("/calculate", methods=["POST"])
def calculate():

    try:

        quantity = float(
            request.form.get(
                "quantity",
                0
            )
        )

        price = float(
            request.form.get(
                "price",
                0
            )
        )


        total = quantity * price


        return f"""
<!DOCTYPE html>
<html>

<head>

    <title>Waste Value Calculator</title>

    <meta name="viewport"
          content="width=device-width, initial-scale=1">

</head>


<body style="
    font-family:Arial;
    text-align:center;
    padding:80px 20px;
    background:#f4f8f2;
    color:#18351f;
">


    <h1>
        💰 Waste Value Calculator
    </h1>


    <div style="
        background:white;
        padding:40px;
        max-width:500px;
        margin:40px auto;
        border-radius:20px;
        box-shadow:0 8px 30px rgba(0,0,0,0.08);
    ">


        <h2>
            Estimated Value
        </h2>


        <p style="
            font-size:36px;
            color:#3c8c45;
            margin:25px;
            font-weight:bold;
        ">

            ₹{total:,.0f}

        </p>


        <p style="
            color:#617064;
            font-size:17px;
        ">

            {quantity:g} tons × ₹{price:,.0f} per ton

        </p>


        <br>


        <a
            href="/"
            style="
                display:inline-block;
                text-decoration:none;
                background:#3c8c45;
                color:white;
                padding:14px 25px;
                border-radius:10px;
                font-size:16px;
            "
        >

            ← Back to Waste2Worth

        </a>


    </div>


</body>

</html>
"""


    except Exception as e:

        return f"""
        <h2>Calculator Error</h2>

        <p>{html.escape(str(e))}</p>

        <a href="/">← Go Back</a>
        """


# ============================================================
# START FLASK
# ============================================================

if __name__ == "__main__":

    app.run(
        debug=True
    )
