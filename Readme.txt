package mx.com.bancoazteca.azteca360.rsdk.view.credit.auth

import android.annotation.SuppressLint
import android.content.BroadcastReceiver
import android.content.Context
import android.content.Intent
import android.content.IntentFilter
import android.content.res.TypedArray
import android.graphics.Typeface
import android.os.CountDownTimer
import android.os.Handler
import android.os.Looper
import android.text.Editable
import android.text.InputFilter
import android.text.TextWatcher
import android.util.AttributeSet
import android.util.Log
import android.view.ActionMode
import android.view.KeyEvent
import android.view.LayoutInflater
import android.view.Menu
import android.view.MenuItem
import android.view.View
import android.view.WindowManager
import android.view.inputmethod.EditorInfo
import android.view.inputmethod.InputConnection
import android.view.inputmethod.InputMethodManager
import android.widget.EditText
import android.widget.FrameLayout
import androidx.annotation.ColorInt
import androidx.appcompat.app.AppCompatActivity
import androidx.core.content.ContextCompat
import androidx.core.content.res.ResourcesCompat
import androidx.core.widget.doAfterTextChanged
import androidx.lifecycle.MutableLiveData
import androidx.localbroadcastmanager.content.LocalBroadcastManager
import mx.com.bancoazteca.azteca360.rsdk.R
import mx.com.bancoazteca.azteca360.rsdk.databinding.BafpCreditLoginOtpBinding

@SuppressLint("CustomViewStyleable")
class BAFPCreditLoginOTP internal constructor(
    context: Context, attrs: AttributeSet? = null
) : FrameLayout(context, attrs) {

    private lateinit var countDownTimer: CountDownTimer
    private var vBind: BafpCreditLoginOtpBinding = BafpCreditLoginOtpBinding.inflate(LayoutInflater.from(context), this, true)
    internal val onNextScreen: MutableLiveData<String?> = MutableLiveData<String?>()
    private lateinit var editTexts: List<EditText>
    private val handler = Handler(Looper.getMainLooper())


//    private val keepKeyboardOpenRunnable = object : Runnable {
//        override fun run() {
//            if (vBind.et1.text.isEmpty()) {
//                vBind.et1.requestFocus()
//                showKeyboard(vBind.et1)
//            }
//            handler.postDelayed(this, 100)
//        }
//    }
private val keepKeyboardOpenRunnable = object : Runnable {
    override fun run() {
        if (!vBind.et1.hasFocus() && vBind.et1.text.isEmpty()) {
            vBind.et1.requestFocus()
            showKeyboard(vBind.et1)
        }
       // handler.postDelayed(this, 100)
    }
}

    private val smsReceiver = object : BroadcastReceiver() {
        override fun onReceive(context: Context, intent: Intent) {
            val code = intent.getStringExtra("code")
            code?.let {
                if (code.length == 7) {
                    for (i in code.indices) {
                        editTexts[i].setText(code[i].toString())
                    }
                    editTexts[6].clearFocus()
                }
            }
        }
    }

    init {
        context.obtainStyledAttributes(attrs, R.styleable.BAFPCreditOtpValidate).let { typedArray ->
            typedArray.setupValues()
            typedArray.recycle()
        }

        if (context is AppCompatActivity) {
            context.window.setFlags(
                WindowManager.LayoutParams.FLAG_SECURE, WindowManager.LayoutParams.FLAG_SECURE
            )
        }
        vBind.et1.requestFocus()
        // Mostrar el teclado virtual
        val imm = context.getSystemService(Context.INPUT_METHOD_SERVICE) as InputMethodManager
        imm.showSoftInput(vBind.et1, InputMethodManager.SHOW_IMPLICIT)

        editTexts = listOf(
            vBind.et1,
            vBind.et2,
            vBind.et3,
            vBind.et4,
            vBind.et5,
            vBind.et6,
            vBind.et7
        )
        initialEvents()
    }
//    override fun onAttachedToWindow() {
//        super.onAttachedToWindow()
//        LocalBroadcastManager.getInstance(context).registerReceiver(smsReceiver,
//            IntentFilter("SMS_CODE_RECEIVED"))
//        vBind.et1.requestFocus()
//        showKeyboard(vBind.et1)
//    }
//    override fun onDetachedFromWindow() {
//        super.onDetachedFromWindow()
//        handler.removeCallbacks(keepKeyboardOpenRunnable)
//    }

    override fun onAttachedToWindow() {
        super.onAttachedToWindow()
        handler.post(keepKeyboardOpenRunnable)
    }

    override fun onDetachedFromWindow() {
        super.onDetachedFromWindow()
        handler.removeCallbacks(keepKeyboardOpenRunnable)
    }
    private fun disableCopyPaste(editText: EditText) {
        editText.setOnLongClickListener { true }
        editText.customSelectionActionModeCallback = object : ActionMode.Callback {
            override fun onCreateActionMode(mode: ActionMode?, menu: Menu?) = false
            override fun onPrepareActionMode(mode: ActionMode?, menu: Menu?) = false
            override fun onActionItemClicked(mode: ActionMode?, item: MenuItem?) = false
            override fun onDestroyActionMode(mode: ActionMode?) {}
        }
        editText.isLongClickable = false
        editText.setTextIsSelectable(false)
    }



    private fun EditText.setupEditText(
        mBinding: BafpCreditLoginOtpBinding,
        next: EditText?,
        prev: EditText?
    ) {
        this.addTextChangedListener(object : TextWatcher {
            override fun beforeTextChanged(s: CharSequence?, start: Int, count: Int, after: Int) {}
            override fun onTextChanged(s: CharSequence?, start: Int, before: Int, count: Int) {}
            override fun afterTextChanged(s: Editable?) {
                if (s?.length == 1) {
                    this@setupEditText.background = ContextCompat.getDrawable(
                        context,
                        R.drawable.custom_rounder_et_border_black
                    )
                    next?.requestFocus()
                } else {
                    this@setupEditText.background =
                        ContextCompat.getDrawable(context, R.drawable.custom_rounder_editext)
                    if (this@setupEditText == mBinding.et1) {
                        mBinding.et1.requestFocus()
                    } else {
                        prev?.requestFocus()
                    }
                }
                updateNextButtonState()
            }
        })

        this.setOnKeyListener { _, keyCode, event ->
            if (keyCode == KeyEvent.KEYCODE_DEL && event.action == KeyEvent.ACTION_DOWN) {
                if (this.text.isEmpty()) {
                    prev?.requestFocus()
                    prev?.text?.clear()
                } else {
                    this.text.clear()
                }
                true
            } else {
                false
            }
        }

        this.filters = arrayOf<InputFilter>(InputFilter.LengthFilter(1))
    }

    private fun setupEditTextListeners() {
        for (i in editTexts.indices) {
            editTexts[i].addTextChangedListener(object : TextWatcher {
                override fun beforeTextChanged(s: CharSequence?, start: Int, count: Int, after: Int) {}
                override fun onTextChanged(s: CharSequence?, start: Int, before: Int, count: Int) {}
                override fun afterTextChanged(s: Editable?) {
                    if (s?.length == 1) {
                        editTexts[i].background = ContextCompat.getDrawable(
                            context,
                            R.drawable.custom_rounder_et_border_black
                        )
                        if (i < editTexts.size - 1) {
                            editTexts[i + 1].isEnabled = true
                            editTexts[i + 1].requestFocus()
                        }
                    } else {
                        editTexts[i].background = ContextCompat.getDrawable(
                            context,
                            R.drawable.custom_rounder_editext
                        )
                    }
                    updateNextButtonState()
                }
            })

            editTexts[i].setOnKeyListener { _, keyCode, event ->
                if (keyCode == KeyEvent.KEYCODE_DEL && event.action == KeyEvent.ACTION_DOWN) {
                    if (editTexts[i].text.isEmpty() && i > 0) {
                        editTexts[i - 1].isEnabled = true
                        editTexts[i - 1].requestFocus()
                        editTexts[i - 1].text?.let { it.delete(it.length - 1, it.length) }
                        return@setOnKeyListener true
                    }
                }
                false
            }
        }
    }

    private fun initialEvents() {
        initTimer(65L)
        vBind.bafpCreditTvTimer.visibility = View.VISIBLE
        for (editText in editTexts) {
            disableCopyPaste(editText)
        }
        vBind.apply {
            bafpCreditTvMessage.text =
                context.getString(R.string.txt_mensaje_envio_cod, "5613363773")
            et1.setupEditText(this, et2, null)
            et2.setupEditText(this, et3, et1)
            et3.setupEditText(this, et4, et2)
            et4.setupEditText(this, et5, et3)
            et5.setupEditText(this, et6, et4)
            et6.setupEditText(this, et7, et5)
            et7.setupEditText(this, null, et6)

            bafpCreditBtnReenviar.setOnClickListener {
                initTimer(65L)
            }
            setupEditTextListeners()
            vBind.et1.requestFocus()
            showKeyboard(vBind.et1)


            btnNext.setOnClickListener {
                onNextScreen.value = getCode()
                context.unregisterReceiver(smsReceiver)
            }
        }

        // Mantener el foco en et1 y el teclado abierto
        vBind.et1.setOnFocusChangeListener { _, hasFocus ->
            if (hasFocus) {
                showKeyboard(vBind.et1)
            }
        }

        // Iniciar el runnable para mantener el teclado abierto
        handler.post(keepKeyboardOpenRunnable)
    }



    private fun updateNextButtonState() {
        vBind.btnNext.isEnabled = editTexts.all { it.text.isNotEmpty() }
    }

    @SuppressLint("UseCompatLoadingForDrawables")
    private fun EditText.validateNumber(
        mBinding: BafpCreditLoginOtpBinding,
        next: EditText?,
        prev: EditText?
    ) {
        val et = this
        doAfterTextChanged {
            disableEt()

            if (it?.isEmpty() == true && et == mBinding.et1) {
                et.postDelayed({
                    et.requestFocus()
                    showKeyboard(et)
                }, 10)
            }
            if (mBinding.et1.text.isNotEmpty() && mBinding.et2.text.isNotEmpty() && mBinding.et3.text.isNotEmpty()
                && mBinding.et4.text.isNotEmpty() && mBinding.et5.text.isNotEmpty() && mBinding.et6.text.isNotEmpty() && mBinding.et7.text.isNotEmpty()) {
                mBinding.btnNext.isEnabled = true
                setBAckroundBlack(mBinding.et1)
                setBAckroundBlack(mBinding.et2)
                setBAckroundBlack(mBinding.et3)
                setBAckroundBlack(mBinding.et4)
                setBAckroundBlack(mBinding.et5)
                setBAckroundBlack(mBinding.et6)
                setBAckroundBlack(mBinding.et7)
            } else {
                mBinding.btnNext.isEnabled = false
            }
            if (it != null) {
                if (it.isNotEmpty()) {
                    et.background = ContextCompat.getDrawable(context, R.drawable.custom_rounder_et_border_black)
                    if (next != null) {
                        next.isEnabled = true
                        next.requestFocus()
                        showKeyboard(next)
                        et.isEnabled = false
                    }
                    if (prev != null) prev.isEnabled = false
                } else {
                    if (et != mBinding.et7) {
                        et.background = ContextCompat.getDrawable(context, R.drawable.custom_rounder_editext)
                        if (et == mBinding.et1) {
                            et.requestFocus()
                            showKeyboard(et)
                        } else {
                            prev?.isEnabled = true
                            prev?.requestFocus()
                            showKeyboard(prev!!)
                            et.text?.clear()
                            if (et != mBinding.et1) {
                                et.isEnabled = false
                            }
                        }
                        mBinding.et7.isEnabled = false
                    } else {
                        et.text?.clear()
                    }
                }
            }
        }
    }
    private fun showKeyboard(view: View) {
        val imm = context.getSystemService(Context.INPUT_METHOD_SERVICE) as InputMethodManager
        imm.showSoftInput(view, InputMethodManager.SHOW_FORCED)
    }

    private fun EditText.other(
        next: EditText?,
        prev: EditText?
    ) {
        this.setOnKeyListener { view, keyCode, event ->
            if (keyCode == KeyEvent.KEYCODE_DEL && this.text.isEmpty() && event.action == KeyEvent.ACTION_DOWN) {
                this.background = ContextCompat.getDrawable(context,R.drawable.custom_rounder_editext)
                prev?.text?.clear()
                return@setOnKeyListener true // Indica que el evento ha sido manejado
            } else if (keyCode == KeyEvent.KEYCODE_DEL && this.text.isNotEmpty() && event.action == KeyEvent.ACTION_DOWN) {
                this.background = ContextCompat.getDrawable(context,R.drawable.custom_rounder_editext)
                this?.text?.clear()
                return@setOnKeyListener true // Indica que el evento ha sido manejado
            } else if(this.text.isNotEmpty() && event.action == KeyEvent.ACTION_DOWN){
                next?.text = Editable.Factory.getInstance().newEditable(event.unicodeChar.toChar().toString())
                return@setOnKeyListener true
            }
            false // Deja que el sistema maneje el evento
        }
    }

    private fun setBAckroundBlack(editText: EditText){
        editText.background = ContextCompat.getDrawable(context,R.drawable.custom_rounder_et_border_black)
    }

    private fun initTimer(seconds: Long){
        vBind.bafpCreditTvQuestion.visibility = View.GONE
        vBind.bafpCreditBtnReenviar.visibility = View.GONE
        vBind.bafpCreditTvTimer.visibility = View.VISIBLE
        countDownTimer = object : CountDownTimer(seconds * 1000, 1000) {

            @SuppressLint("DefaultLocale")
            override fun onTick(millisUntilFinished: Long) {
                val secondsRemaining = millisUntilFinished / 1000
                val secondsDigits = String.format("%02d", secondsRemaining - (60*(secondsRemaining/60)))
                when (secondsRemaining) {
                    in 0..seconds -> vBind.bafpCreditTvTimer.text = context.getString(
                        R.string.txt_timer,
                        "${secondsRemaining/60}:${secondsDigits}"
                    )
                }
            }
            override fun onFinish() {
                vBind.bafpCreditTvQuestion.visibility = View.VISIBLE
                vBind.bafpCreditBtnReenviar.visibility = View.VISIBLE
                vBind.bafpCreditTvTimer.visibility = View.GONE
            }
        }
        countDownTimer.start()
    }

    private fun getCode():String{
        return "${vBind.et1.text}${vBind.et2.text}${vBind.et3.text}${vBind.et4.text}${vBind.et5.text}${vBind.et6.text}${vBind.et7.text}"
    }


    private fun disableEt(){
        vBind.apply {
            if (et1.text.isEmpty() && et2.text.isEmpty() && et3.text.isEmpty() && et4.text.isEmpty() && et5.text.isEmpty() && et6.text.isEmpty() && et7.text.isEmpty()){
                et2.isEnabled = false
                et3.isEnabled = false
                et4.isEnabled = false
                et5.isEnabled = false
                et6.isEnabled = false
                et7.isEnabled = false
            }
        }
    }



    @SuppressLint("ResourceType")
    private fun TypedArray.setupValues() {
        setFontFamilyTittle(getString(R.styleable.BAFPCreditOtpPhone_setFontFamilyTitleCreditPhone))
        setFontFamilySubtitle( getString(R.styleable.BAFPCreditOtpPhone_setFontFamilyMessageCreditPhone))
        setFontFamilyTxtQuestion(getString(R.styleable.BAFPCreditOtpValidate_setFontFamilyQuestionOtpValidate))
        setFontFamilyTxtBtnReenviar(getString(R.styleable.BAFPCreditOtpValidate_setFontFamilyTxtBtnReenviarOtpValidate))
        setFontFamilyTxtTimer( getString(R.styleable.BAFPCreditOtpValidate_setFontFamilyTxtTimerOtpValidate))
        setColorTextTitle(getColor(R.styleable.BAFPCreditOtpPhone_setColorTextTitleCreditPhone, ContextCompat.getColor(context, R.color.background_button_red)))
        setColorTextMessage(getColor(R.styleable.BAFPCreditOtpPhone_setColorTextMessageCreditPhone, ContextCompat.getColor(context, R.color.Black)))
        setColorTextTxtQuestion(getColor(R.styleable.BAFPCreditOtpValidate_setColorTextQuestionOtpValidate, ContextCompat.getColor(context, R.color.Black)))
        setColorTextTxtBtnReenviar(getColor(R.styleable.BAFPCreditOtpValidate_setColorTextBtnReenviarOtpValidate, ContextCompat.getColor(context, R.color.bafp_credit_btn_blue)))
        setColorTextTimer(getColor(R.styleable.BAFPCreditOtpValidate_setColorTextTxtTimerOtpValidate, ContextCompat.getColor(context, R.color.Black)))
    }

    fun setFontFamilyTittle(fontFamily: String?){
        if (fontFamily == null) {
            vBind.bafpCreditTvTittle.typeface = ResourcesCompat.getFont(context, R.font.roboto_medium)
        }else{
            vBind.bafpCreditTvTittle.typeface = Typeface.createFromAsset(context.assets, fontFamily)
        }
    }
    fun setFontFamilySubtitle(fontFamily: String?){
        if (fontFamily == null) {
            vBind.bafpCreditTvMessage.typeface = ResourcesCompat.getFont(context, R.font.roboto_medium)
        }else{
            vBind.bafpCreditTvMessage.typeface = Typeface.createFromAsset(context.assets, fontFamily)
        }
    }
    fun setColorTextTitle(@ColorInt color: Int?){
        if (color == null){
            vBind.bafpCreditTvTittle.setTextColor(ContextCompat.getColor(context,R.color.background_button_red))
        }else{
            vBind.bafpCreditTvTittle.setTextColor(color)
        }
    }
    fun setColorTextMessage(@ColorInt color: Int?){
        if (color == null){
            vBind.bafpCreditTvMessage.setTextColor(ContextCompat.getColor(context,R.color.Black))
        }else{
            vBind.bafpCreditTvMessage.setTextColor(color)
        }
    }
    fun setFontFamilyTxtQuestion(fontFamily: String?){
        if (fontFamily == null){
            vBind.bafpCreditTvQuestion.typeface = ResourcesCompat.getFont(context, R.font.roboto_medium)
        }else{
            vBind.bafpCreditTvQuestion.typeface = Typeface.createFromAsset(context.assets, fontFamily)
        }
    }
    fun setColorTextTxtQuestion(@ColorInt color: Int?){
        if (color == null){
            vBind.bafpCreditTvQuestion.setTextColor(ContextCompat.getColor(context,R.color.Black))
        }else{
            vBind.bafpCreditTvQuestion.setTextColor(color)
        }
    }
    fun setFontFamilyTxtBtnReenviar(fontFamily: String?){
        if (fontFamily == null){
            vBind.bafpCreditBtnReenviar.typeface = ResourcesCompat.getFont(context, R.font.roboto_medium)
        }else{
            vBind.bafpCreditBtnReenviar.typeface = Typeface.createFromAsset(context.assets, fontFamily)
        }
    }
    fun setColorTextTxtBtnReenviar(@ColorInt color: Int?){
        if (color == null){
            vBind.bafpCreditBtnReenviar.setTextColor(ContextCompat.getColor(context,R.color.bafp_credit_btn_blue))
        }else{
            vBind.bafpCreditBtnReenviar.setTextColor(color)
        }
    }
    fun setFontFamilyTxtTimer(fontFamily: String?){
        if (fontFamily == null){
            vBind.bafpCreditTvTimer.typeface = ResourcesCompat.getFont(context, R.font.roboto_medium)
        }else{
            vBind.bafpCreditTvTimer.typeface = Typeface.createFromAsset(context.assets, fontFamily)
        }
    }
    fun setColorTextTimer(@ColorInt color: Int?){
        if (color == null){
            vBind.bafpCreditTvTimer.setTextColor(ContextCompat.getColor(context,R.color.Black))
        }else{
            vBind.bafpCreditTvTimer.setTextColor(color)
        }
    }
}
