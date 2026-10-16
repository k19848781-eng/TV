package com.pishangay.tv

import android.app.Activity
import android.graphics.Color
import android.os.Bundle
import android.view.View
import android.widget.*
import androidx.media3.common.MediaItem
import androidx.media3.common.PlaybackException
import androidx.media3.common.Player
import androidx.media3.datasource.DefaultHttpDataSource
import androidx.media3.exoplayer.ExoPlayer
import androidx.media3.exoplayer.source.DefaultMediaSourceFactory
import androidx.media3.ui.PlayerView
import java.net.HttpURLConnection
import java.net.URL
import kotlin.concurrent.thread

class MainActivity : Activity() {
    private val playlist = "https://raw.githubusercontent.com/k19848781-eng/Mohammed/main/playlist.m3u"
    private lateinit var exo: ExoPlayer
    private val names = ArrayList<String>()
    private val urls = ArrayList<String>()

    override fun onCreate(b: Bundle?) {
        super.onCreate(b)
        val http = DefaultHttpDataSource.Factory()
            .setUserAgent("VLC/3.0.18 LibVLC/3.0.18").setAllowCrossProtocolRedirects(true)
        exo = ExoPlayer.Builder(this).setMediaSourceFactory(DefaultMediaSourceFactory(http)).build()

        val root = LinearLayout(this).apply {
            orientation = LinearLayout.VERTICAL
            setBackgroundColor(Color.parseColor("#111A22"))
            layoutDirection = View.LAYOUT_DIRECTION_RTL
        }
        val title = TextView(this).apply {
            text = "پێشانگای محمد جبار لە سیدسادق"
            setTextColor(Color.WHITE); textSize = 20f; setPadding(32, 48, 32, 16)
        }
        val pv = PlayerView(this).apply {
            player = exo
            layoutParams = LinearLayout.LayoutParams(-1, (240 * resources.displayMetrics.density).toInt())
        }
        val status = TextView(this).apply {
            text = "بارکردنی کەناڵەکان…"; setTextColor(Color.LTGRAY); setPadding(32, 12, 32, 12)
        }
        val list = ListView(this)
        val adapter = ArrayAdapter(this, android.R.layout.simple_list_item_1, names)
        list.adapter = adapter
        root.addView(title); root.addView(pv); root.addView(status); root.addView(list)
        setContentView(root)

        list.setOnItemClickListener { _, _, i, _ ->
            status.text = "بارکردنی " + names[i] + "…"
            exo.setMediaItem(MediaItem.fromUri(urls[i]))
            exo.prepare(); exo.playWhenReady = true
        }
        exo.addListener(object : Player.Listener {
            override fun onPlaybackStateChanged(s: Int) {
                if (s == Player.STATE_READY) status.text = "▶ " + (names.getOrNull(list.checkedItemPosition) ?: "")
            }
            override fun onPlayerError(e: PlaybackException) {
                status.text = "ئەم کەناڵە کار ناکات. کەناڵێکی تر تاقی بکەرەوە."
            }
        })

        thread {
            try {
                val c = URL(playlist).openConnection() as HttpURLConnection
                c.connectTimeout = 15000; c.readTimeout = 15000
                val lines = c.inputStream.bufferedReader().readLines()
                var n: String? = null
                val seen = HashSet<String>()
                val nn = ArrayList<String>(); val uu = ArrayList<String>()
                for (raw in lines) {
                    val l = raw.trim()
                    if (l.startsWith("#EXTINF")) n = l.substringAfterLast(',').trim()
                    else if (l.isNotEmpty() && !l.startsWith("#") && n != null) {
                        if (seen.add(l)) { nn.add(n!!); uu.add(l) }
                        n = null
                    }
                }
                runOnUiThread {
                    names.addAll(nn); urls.addAll(uu); adapter.notifyDataSetChanged()
                    status.text = "کەناڵێک هەڵبژێرە (" + nn.size + ")"
                }
            } catch (e: Exception) {
                runOnUiThread { status.text = "پلەیلیست بار نەبوو. ئینتەرنێتەکەت بپشکنە." }
            }
        }
    }

    override fun onStop() { super.onStop(); exo.pause() }
    override fun onDestroy() { super.onDestroy(); exo.release() }
}
