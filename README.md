# How-to-POST-curl-Type-API
I have read number of post regarding this problem, finally i m successful in solving curl type post api. Really this was challenging work for me.

Step 1 : This is crul POST Type api,

    
      curl -v -S -u devsmart:abc@123 
      F'notification{"applicationId":"32770","schemaId":"32787","topicId":"32770","type":"USER"};
            type=application/json' -F file=@/home/ananya/Desktop/notification.json "http://13.126.140.6:8080/kaaAdmin/rest/api/sendNotification" | python -mjson.tool
    
    
Step 2 :Add this library in your Module app gradle. Below library use for POST file with crul api. 

    implementation'org.apache.httpcomponents:httpmime:4.3'

Here i goning to explain some notation which used in crul type api, F notation denote for File, so we need to create a JSON file, in my case i had created notification.json file in this file we have put {"applicationId":"32770","schemaId":"32787","topicId":"32770","type":"USER"}.


Step 3: Here the pasring code FOR CRUL TYPE API.

    import android.app.ProgressDialog;
    import android.os.AsyncTask;
    import android.os.Environment;
    import android.os.Handler;
    import android.support.v7.app.AppCompatActivity;
    import android.os.Bundle;
    import android.util.Base64;
    import android.util.Log;
    import android.view.View;
    import android.widget.CompoundButton;
    import android.widget.EditText;
    import android.widget.Switch;
    import android.widget.Toast;

    import com.android.volley.AuthFailureError;
    import com.android.volley.DefaultRetryPolicy;
    import com.android.volley.Request;
    import com.android.volley.RequestQueue;
    import com.android.volley.Response;
    import com.android.volley.VolleyError;
    import com.android.volley.toolbox.JsonObjectRequest;
    import com.android.volley.toolbox.Volley;

    import org.apache.http.HttpEntity;
    import org.apache.http.HttpResponse;
    import org.apache.http.HttpVersion;
    import org.apache.http.NameValuePair;
    import org.apache.http.auth.UsernamePasswordCredentials;
    import org.apache.http.client.ClientProtocolException;
    import org.apache.http.client.HttpClient;
    import org.apache.http.client.entity.UrlEncodedFormEntity;
    import org.apache.http.client.methods.HttpPost;
    import org.apache.http.conn.ClientConnectionManager;
    import org.apache.http.conn.scheme.PlainSocketFactory;
    import org.apache.http.conn.scheme.Scheme;
    import org.apache.http.conn.scheme.SchemeRegistry;
    import org.apache.http.conn.ssl.SSLSocketFactory;
    import org.apache.http.entity.mime.MultipartEntity;
    import org.apache.http.entity.mime.content.ContentBody;
    import org.apache.http.entity.mime.content.FileBody;
    import org.apache.http.entity.mime.content.StringBody;
    import org.apache.http.impl.auth.BasicScheme;
    import org.apache.http.impl.client.DefaultHttpClient;
    import org.apache.http.impl.conn.tsccm.ThreadSafeClientConnManager;
    import org.apache.http.message.BasicNameValuePair;
    import org.apache.http.params.BasicHttpParams;
    import org.apache.http.params.HttpParams;
    import org.apache.http.params.HttpProtocolParams;
    import org.apache.http.protocol.HTTP;
    import org.json.JSONException;
    import org.json.JSONObject;

    import java.io.BufferedReader;
    import java.io.DataOutputStream;
    import java.io.File;
    import java.io.FileWriter;
    import java.io.IOException;
    import java.io.InputStream;
    import java.io.InputStreamReader;
    import java.io.UnsupportedEncodingException;
    import java.net.HttpURLConnection;
    import java.net.URI;
    import java.net.URL;
    import java.nio.charset.Charset;
    import java.security.KeyStore;
    import java.util.ArrayList;
    import java.util.HashMap;
    import java.util.List;
    import java.util.Map;

    public class MainActivity extends AppCompatActivity {
        ProgressDialog PD;
        Switch aSwitch;

    String nmbr="1";
    EditText editText;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        editText=findViewById(R.id.nmbr);

        nmbr=editText.getText().toString();
        aSwitch=findViewById(R.id.swtch);
        aSwitch.setOnCheckedChangeListener(new CompoundButton.OnCheckedChangeListener() {
            @Override
            public void onCheckedChanged(CompoundButton compoundButton, boolean b) {
                PD = new ProgressDialog(MainActivity.this,5);
                PD.setCancelable(false);
                PD.setTitle("Sending Data");
                PD.setMessage("Sending data please wait");
                PD.show();
                if (b){
                
                // This function for put on value in my JSON file.
                
                    generateNoteOnSD("on",nmbr);
                    // This function for execute PostData API
                    new PostData().execute();
                }
                else {
                    generateNoteOnSD("off",nmbr);
                    new PostData().execute();

                }
            }
        });

}

// Here is the POST api parsing code  

    private class PostData extends AsyncTask<String, String, String> {
        HttpPost httpost = new HttpPost();
        MultipartEntity mpEntity = new MultipartEntity();
        BufferedReader in = null;
        StringBuffer sb = new StringBuffer();
        BufferedReader inPost = null;


        @Override
        protected void onPreExecute() {
            try{



                ContentBody notification = new StringBody(getJsonData().toString(), "application/json", Charset.forName("UTF-8"));
                mpEntity.addPart("notification", notification);

                ContentBody file = new FileBody(new File("/storage/emulated/0/CreateFiles/Report Files/notification.json"));
                mpEntity.addPart("file", file);


            }catch (Exception e){
                Log.e("errpree",e.toString());


            }
        }
        @Override
        protected String doInBackground(String... params) {
            try{


                DefaultHttpClient httpclient = new DefaultHttpClient();


                URI uri=new URI("http://13.126.140.6:8080/kaaAdmin/rest/api/sendNotification");
                httpost.setURI(uri);

                httpost.addHeader(BasicScheme.authenticate(
                        new UsernamePasswordCredentials("devsmart", "abc@123"),
                        HTTP.UTF_8, false));


                httpost.setEntity(mpEntity);

                HttpResponse response = httpclient.execute(httpost);
                HttpEntity entity = response.getEntity();

                inPost = new BufferedReader(new InputStreamReader(entity.getContent()));
                String linePost = "";
                String NLPOST = System.getProperty("line.separator");
                while ((linePost = inPost.readLine()) != null) {
                    sb.append(linePost + NLPOST);
                }
                inPost.close();


                if (entity != null) {
                    Log.e("empty","entity");
                    entity.consumeContent();
                }

                return sb.toString();



            } catch (Exception e) {
                Log.e("err",e.toString());
                e.printStackTrace();
            }


            return null;
        }

        @Override
        protected void onPostExecute(String s) {
            PD.dismiss();
            try{

                Log.e("response",s.toString());

            }catch (Exception e){
                Log.e("errpost",e.toString());

            }
        }


    }

    public JSONObject getJsonData(){

        JSONObject jsonobject=new JSONObject();
        try {
            jsonobject.put("applicationId", "32770");
            jsonobject.put("schemaId", "32787");
            jsonobject.put("topicId","32770");
            jsonobject.put("type","USER");




        }catch (JSONException e) {
            e.printStackTrace();
        }
        return jsonobject;
    }



    public void generateNoteOnSD(String state,String brightness) {

        String fileName =  "notification.json";


        try {
            File root = new File(Environment.getExternalStorageDirectory() + File.separator + "CreateFiles", "Report Files");

            if (!root.exists()) {
                root.mkdirs();
            }
            File gpxfile = new File(root, fileName);


            FileWriter writer = new FileWriter(gpxfile, false);



            writer.write("{\"State\":{\"string\": \""+state+"\"},\"Brightness\":{\"string\": \""+brightness+"\"}}\n");
            writer.flush();
            writer.close();


            Toast.makeText(this, root.toString(), Toast.LENGTH_SHORT).show();
        } catch (Exception e) {
            Log.e("err",e.toString());

        }
         }


    }


