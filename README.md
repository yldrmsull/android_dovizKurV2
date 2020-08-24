# android_dovizKurV2


# Custom listView'imizi oluşturmak için kur.java ve CustomAdapter.java dosyalarımızı oluşturuyoruz. Amacımız apiden çekmiş olduğumuz verileri customListView'imizde göstermek.
# layout dosyalarını yukarıda paylaştım.
# kur
```java

public class kur {
    private String paraBirimiAlis;
    private String paraBirimiSatis;
    private String paraBirimi;
    private boolean paraArtis;
    private String time;
    private String rate;


    public kur(String paraBirimi,String time, String paraBirimiAlis, String paraBirimiSatis, boolean paraArtis,String rate) {
        this.paraBirimiAlis = paraBirimiAlis;
        this.paraBirimiSatis = paraBirimiSatis;
        this.paraArtis=paraArtis;
        this.paraBirimi=paraBirimi;
        this.time=time;
        this.rate=rate;
    }
    public String getParaBirimiAlis() {
        return paraBirimiAlis;
    }

    public void setParaBirimiAlis(String paraBirimiAlis) {
        this.paraBirimiAlis = paraBirimiAlis;
    }

    public String getParaBirimiSatis() {
        return paraBirimiSatis;
    }

    public void setParaBirimiSatis(String paraBirimiSatis) {
        this.paraBirimiSatis = paraBirimiSatis;
    }

    public String getParaBirimi() {
        return paraBirimi;
    }

    public void setParaBirimi(String paraBirimi) {
        this.paraBirimi = paraBirimi;
    }

    public boolean isParaArtis() {
        return paraArtis;
    }

    public void setParaArtis(boolean paraArtis) {
        this.paraArtis = paraArtis;
    }

    public String getTime() {
        return time;
    }

    public void setTime(String time) {
        this.time = time;
    }

    public String getRate() {
        return rate;
    }

    public void setRate(String rate) {
        this.rate = rate;
    }
}
```
# CustomAdapter
```java
public class CustomAdapter extends BaseAdapter {
    private LayoutInflater kurInflater;
    private List<kur> kurList;

    public CustomAdapter(Activity activity, List<kur> kurList) {
        kurInflater = (LayoutInflater) activity.getSystemService(
                Context.LAYOUT_INFLATER_SERVICE);
        this.kurList = kurList;
    }

    @Override
    public int getCount() {
        return kurList.size();
    }

    @Override
    public Object getItem(int i) {
        return kurList.get(i);
    }

    @Override
    public long getItemId(int i) {
        return i;
    }

    @Override
    public View getView(int i, View view, ViewGroup viewGroup) {
        View lineView;
        lineView = kurInflater.inflate(R.layout.activity_custom_listview, null);
        TextView textViewParaBirimi = (TextView) lineView.findViewById(R.id.tvParaBirimi);
        TextView textViewParaAlis = (TextView) lineView.findViewById(R.id.tvParaAlis);
        TextView textViewParaSatis = (TextView) lineView.findViewById(R.id.tvParaSatis);
        TextView textViewTime=(TextView) lineView.findViewById(R.id.tvTime);
        TextView textViewRate=(TextView) lineView.findViewById(R.id.tvRate);
        ImageView imageViewRatePicture = (ImageView) lineView.findViewById(R.id.imvParaArtis);
        kur k = kurList.get(i);
        textViewParaBirimi.setText(k.getParaBirimi());
        textViewParaAlis.setText(k.getParaBirimiAlis());
        textViewParaSatis.setText(k.getParaBirimiSatis());
        textViewTime.setText(k.getTime());
        textViewRate.setText(k.getRate());

        if (k.isParaArtis()) {
            imageViewRatePicture.setImageResource(R.mipmap.up_icon);
        } else {
            imageViewRatePicture.setImageResource(R.mipmap.down_icon);
        }
        return lineView;
    }
}
```
# Şimdi sıra api bağlantımızı yapmakta.
# MainActivity
```java
public class MainActivity extends AppCompatActivity {
    CountDownTimer countDownTimer;
    int yenilemeSure=4000;
    final List<kur> kurs = new ArrayList<>();

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_deneme);
        final ListView listView = findViewById(R.id.listView);
        final CustomAdapter adapter = new CustomAdapter(this, kurs);
        listView.setAdapter(adapter);

        try {
            adapter.notifyDataSetChanged();
            listView.setAdapter(adapter);
            CheckNews news = new CheckNews();
            news.execute();
            countDownTimer = new CountDownTimer(yenilemeSure, 1000) {
                @Override
                public void onTick(long l) {
                }
                @Override
                public void onFinish() {
                    adapter.notifyDataSetChanged();
                    listView.setAdapter(adapter);
                }
            }.start();
        }
        catch (Exception e) {
            e.printStackTrace();
            System.out.println("hata1");
        }
        Button b =findViewById(R.id.button);
        b.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                adapter.notifyDataSetChanged();
                listView.setAdapter(adapter);
            }
        });
    }
     class CheckNews extends AsyncTask<String,String,String> {
        @Override
        protected String doInBackground(String ... params) {
            String result = new String();
            try {
                result += veriCekme();
            } catch (IOException e) {
                Log.e("Hata", e.getLocalizedMessage());
            }
            return result;
        }
        @Override
        protected void onPreExecute() {
            super.onPreExecute();
        }
        protected void onPostExecute(String s) {
            try{
                String jsonString =s;
                float rate;
                JSONObject obj = new JSONObject(jsonString);
                JSONArray arr = obj.getJSONArray("result");
                for (int i = 0; i < 15; i++)
                {
                    rate=Float.parseFloat((arr.getJSONObject(i).getString("rate")));
                    if(rate>0){
                        kurs.add(new kur(arr.getJSONObject(i).getString("code"),arr.getJSONObject(i).getString("time"), arr.getJSONObject(i).getString("buying"),arr.getJSONObject(i).getString("selling"),true,arr.getJSONObject(i).getString("rate")));
                }
                    else
                        kurs.add(new kur(arr.getJSONObject(i).getString("code"),arr.getJSONObject(i).getString("time"), arr.getJSONObject(i).getString("buying"),arr.getJSONObject(i).getString("selling"),false,arr.getJSONObject(i).getString("rate")));
                }
            }
            catch (Exception e){
                e.printStackTrace();
            }
        }
        public String veriCekme() throws IOException {
            URL url = new URL("https://api.collectapi.com/economy/allCurrency");
            URLConnection urlConnection = url.openConnection();
            urlConnection.setRequestProperty("content-type", "application/json");
            urlConnection.setRequestProperty("authorization", "apikeyimizi buraya yazıyoruz.");          
            BufferedReader br = new BufferedReader(new InputStreamReader(
                    urlConnection.getInputStream()));
            String result = br.readLine();
            br.close();           
            return result;
        }
    }
}
```
# Burası bağlantımızı yaptığımız kısım:
```java
public String veriCekme() throws IOException {
            URL url = new URL("https://api.collectapi.com/economy/allCurrency");
            URLConnection urlConnection = url.openConnection();
            urlConnection.setRequestProperty("content-type", "application/json");
            urlConnection.setRequestProperty("authorization", "apikeyimizi buraya yazıyoruz.");          
            BufferedReader br = new BufferedReader(new InputStreamReader(
                    urlConnection.getInputStream()));
            String result = br.readLine();
            br.close();           
            return result;
        }
```
# Program bu şekilde:

![Screenshot_2020-08-24-03-23-36-691_com example dovizkur](https://user-images.githubusercontent.com/36545904/90992615-9cc3a780-e5b9-11ea-8200-4ebee25d365e.jpg)




