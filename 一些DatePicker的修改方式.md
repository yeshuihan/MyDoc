```java
public class DatePickerUtil {
    private static final String TAG = "DatePickerUtil";
    private Context mContext;
    private OnSelectedDateListener mListener;
 
    public interface OnSelectedDateListener {
        void onSelectedDate(long date);
 
    }
 
    public DatePickerUtil(Context context, OnSelectedDateListener listener) {
        mContext = context;
        mListener = listener;
    }
 
    public void showDatePicker(long showTime) {
        View view = LayoutInflater.from(mContext).inflate(R.layout.dialog_choose_birthday, null);
 
 
        Button confirm = view.findViewById(R.id.dialog_birthday_choose_confirm);
        final Button cancel = view.findViewById(R.id.dialog_birthday_choose_cancel);
        final DatePicker datePicker = view.findViewById(R.id.dialog_birthday_choose_datepicker);
        final Calendar nowtime = Calendar.getInstance();
        nowtime.setTimeInMillis(System.currentTimeMillis());
        final Calendar minCalendar = Calendar.getInstance();
        minCalendar.set(1901,0,1);  //设置最小时间为1901-01-01。默认的最小时间为1900-01-01，但是在拉动到最小的时候会出现异常，导致报停
        datePicker.init(nowtime.get(Calendar.YEAR),
                nowtime.get(Calendar.MONTH),
                nowtime.get(Calendar.DAY_OF_MONTH),
                new DatePicker.OnDateChangedListener() {
            @Override
            public void onDateChanged(DatePicker view, int year, int monthOfYear, int dayOfMonth) {
                changeDatePicker(view, nowtime,minCalendar, year, monthOfYear, dayOfMonth);
            }
        });
        datePicker.setMaxDate(System.currentTimeMillis());
        datePicker.setMinDate(minCalendar.getTimeInMillis());
        changeDatePicker(datePicker, nowtime,minCalendar, nowtime.get(Calendar.YEAR), nowtime.get(Calendar.MONTH), nowtime.get(Calendar.DAY_OF_MONTH));
 
        Calendar calendar = Calendar.getInstance();
        calendar.setTime(new Date(showTime));
        datePicker.updateDate(calendar.get(Calendar.YEAR), calendar.get(Calendar.MONTH), calendar.get(Calendar.DAY_OF_MONTH));
 
        changeDatePickerDivider(datePicker);
        final AlertDialog.Builder builder = new AlertDialog.Builder(mContext);
        builder.setView(view);
        final AlertDialog dialog = builder.create();
 
        confirm.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                datePicker.clearFocus();
                int year = datePicker.getYear();
                int month = datePicker.getMonth() + 1;
                int dayOfMonth = datePicker.getDayOfMonth();
                String dateStr = year + "-" + month + "-" + dayOfMonth;
 
                try {
                    long date = Util.timeStringToLong(dateStr, Config.DATE_FORMATE_TYPE);
                    mListener.onSelectedDate(date);
                } catch (ParseException e) {
                    e.printStackTrace();
                }
                dialog.dismiss();
            }
        });
        cancel.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                dialog.dismiss();
            }
        });
        dialog.show();
        //下面的主要是为了控制dialog的布局
        Window window = dialog.getWindow();
        if (window != null) {
            WindowManager.LayoutParams params = window.getAttributes();
            //params.width = mContext.getResources().getDimensionPixelOffset(R.dimen.birthday_choose_width);
            params.height = WindowManager.LayoutParams.WRAP_CONTENT;
            window.setAttributes(params);
        }
 
    }
    //这里主要是为了更改datepicker的横线的颜色
    private void changeDatePickerDivider(DatePicker datePicker) {
        Resources systemResources = Resources.getSystem();
        try {
            int monthNumberPickerId = systemResources.getIdentifier("month", "id", "android");
            int dayNumberPickerId = systemResources.getIdentifier("day", "id", "android");
            int yearNumberPickerId = systemResources.getIdentifier("year", "id", "android");
            NumberPicker monthNumberPicker = (NumberPicker) datePicker.findViewById(monthNumberPickerId);
            NumberPicker dayNumberPicker = (NumberPicker) datePicker.findViewById(dayNumberPickerId);
            NumberPicker yearNumberPicker = (NumberPicker) datePicker.findViewById(yearNumberPickerId);
            if (monthNumberPicker ==null || dayNumberPicker == null || yearNumberPicker ==null) {
                KLog.i(TAG,"changeDatePickerDivider failed : some NumberPicker is null");
                return;
            }
            //这里设置监听器是为了防止mInputText 被设置为Visible,从而导致日期数字不显示，参考NumberPicker.ondraw()
            dayNumberPicker.setDescendantFocusability(NumberPicker.FOCUS_BLOCK_DESCENDANTS);
            dayNumberPicker.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    //KLog.i(TAG,"changeDatePickerDivider setOnClickListener");
                }
            });
            dayNumberPicker.setOnLongClickListener(new View.OnLongClickListener() {
                @Override
                public boolean onLongClick(View v) {
                    return true;
                }
            });
            monthNumberPicker.setDescendantFocusability(NumberPicker.FOCUS_BLOCK_DESCENDANTS);
            yearNumberPicker.setDescendantFocusability(NumberPicker.FOCUS_BLOCK_DESCENDANTS);
            setNumberPickerDivider(monthNumberPicker);
            setNumberPickerDivider(dayNumberPicker);
            setNumberPickerDivider(yearNumberPicker);
        }catch (Exception e){
            KLog.i(TAG,"changeDatePickerDivider failed:"+e);
        }
    }
    //最小值之前与最大值之后的数字不可选
    private void changeDatePicker(DatePicker datePicker, Calendar maxDate,Calendar minDate, int year, int monthOfYear, int dayOfMonth) {
        Resources systemResources = Resources.getSystem();
        try {
            int monthNumberPickerId = systemResources.getIdentifier("month", "id", "android");
            int dayNumberPickerId = systemResources.getIdentifier("day", "id", "android");
            int yearNumberPickerId = systemResources.getIdentifier("year", "id", "android");
            NumberPicker monthNumberPicker = (NumberPicker) datePicker.findViewById(monthNumberPickerId);
            NumberPicker dayNumberPicker = (NumberPicker) datePicker.findViewById(dayNumberPickerId);
            NumberPicker yearNumberPicker = (NumberPicker) datePicker.findViewById(yearNumberPickerId);
            if (monthNumberPicker ==null || dayNumberPicker == null || yearNumberPicker ==null) {
                KLog.i(TAG,"changeDatePicker failed : some NumberPicker is null");
                return;
            }
            yearNumberPicker.setMaxValue(maxDate.get(Calendar.YEAR));
            if (year == maxDate.get(Calendar.YEAR)) {
                if (monthOfYear == maxDate.get(Calendar.MONTH)) {
                    if (dayOfMonth == maxDate.get(Calendar.DAY_OF_MONTH)) {
                        setNickPickerInputTextInvisible(dayNumberPicker);
                        dayNumberPicker.setValue(dayOfMonth - 1);
                        dayNumberPicker.setValue(dayOfMonth);
                        dayNumberPicker.setMinValue(maxDate.getActualMinimum(Calendar.DAY_OF_MONTH));
                        dayNumberPicker.setMaxValue(maxDate.get(Calendar.DAY_OF_MONTH));
                        dayNumberPicker.setWrapSelectorWheel(true);
                        dayNumberPicker.setWrapSelectorWheel(false);
                    }
                    monthNumberPicker.setValue(monthOfYear - 1);
                    monthNumberPicker.setValue(monthOfYear);
                    monthNumberPicker.setMinValue(maxDate.getActualMinimum(Calendar.MONTH));
                    monthNumberPicker.setMaxValue(maxDate.get(Calendar.MONTH));
                    monthNumberPicker.setWrapSelectorWheel(false);
                }
            } else if (year == minDate.get(Calendar.YEAR)) {
                if (monthOfYear == minDate.get(Calendar.MONTH)) {
                    if (dayOfMonth == minDate.get(Calendar.DAY_OF_MONTH)) {
                        setNickPickerInputTextInvisible(dayNumberPicker);
                        dayNumberPicker.setValue(dayOfMonth + 1);
                        dayNumberPicker.setValue(dayOfMonth);
                        dayNumberPicker.setMinValue(minDate.get(Calendar.DAY_OF_MONTH));
                        dayNumberPicker.setMaxValue(minDate.getActualMaximum(Calendar.DAY_OF_MONTH));
                        dayNumberPicker.setWrapSelectorWheel(true);
                        dayNumberPicker.setWrapSelectorWheel(false);
                    }
                }
            }
 
        }catch (Exception e){
            KLog.i(TAG,"changeDatePicker failed:"+e);
        }
    }
 
    /**
     * 设置NumberPicker.mInputText为Invisible
     * @param numberPicker
     */
    private void setNickPickerInputTextInvisible(NumberPicker numberPicker){
        try {
            Field field = numberPicker.getClass().getDeclaredField("mInputText");
            field.setAccessible(true);
            View mInputText= (View) field.get(numberPicker);
            mInputText.setVisibility(View.INVISIBLE);
        } catch (NoSuchFieldException | IllegalAccessException | IllegalArgumentException e) {
            e.printStackTrace();
        }
    }
 
    private void setNumberPickerDivider(NumberPicker numberPicker) {
        try {
            Field dividerField = numberPicker.getClass().getDeclaredField("mSelectionDivider");
            dividerField.setAccessible(true);
            ColorDrawable colorDrawable = new ColorDrawable(mContext.getResources().getColor(R.color.date_picker_line_color));
            dividerField.set(numberPicker, colorDrawable);
            numberPicker.invalidate();
 
            Field dividerHeightField = numberPicker.getClass().getDeclaredField("mSelectionDividerHeight");
            dividerHeightField.setAccessible(true);
            dividerHeightField.set(numberPicker, mContext.getResources().getDimensionPixelOffset(R.dimen.date_picker_line_height));
        } catch (NoSuchFieldException | IllegalAccessException | IllegalArgumentException e) {
            e.printStackTrace();
        }
    }
 
}
```



