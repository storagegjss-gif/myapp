from flask import Flask, request, jsonify
import yt_dlp, time, json, os, threading

app = Flask(__name__)

CACHE_FILE = "cache.json"
EXPIRY = 3600  # 1 hour
LOCK = threading.Lock()

def load_cache():
    if not os.path.exists(CACHE_FILE):
        return {}
    with open(CACHE_FILE, "r") as f:
        return json.load(f)

def save_cache(data):
    with open(CACHE_FILE, "w") as f:
        json.dump(data, f)

def generate_m3u8(url):
    ydl_opts = {"quiet": True, "skip_download": True}
    with yt_dlp.YoutubeDL(ydl_opts) as ydl:
        info = ydl.extract_info(url, download=False)

    links = {}
    for f in info.get("formats", []):
        if f.get("protocol") in ("m3u8", "m3u8_native"):
            key = f.get("resolution") or f.get("format_id")
            links[key] = f.get("url")

    return {"title": info.get("title"), "links": links}

@app.route("/get")
def get_video():
    yt_url = request.args.get("url")
    if not yt_url:
        return jsonify({"error": "youtube url missing"}), 400

    now = time.time()

    with LOCK:
        cache = load_cache()
        data = cache.get(yt_url)

        if data and now - data["time"] < EXPIRY:
            return jsonify({"source": "cache", "data": data["result"]})

        result = generate_m3u8(yt_url)
        cache[yt_url] = {"time": now, "result": result}
        save_cache(cache)

        return jsonify({"source": "generated", "data": result})

if __name__ == "__main__":
    port = int(os.environ.get("PORT", 3000))
    app.run(host="0.0.0.0", port=port)
