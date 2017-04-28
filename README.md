# Codigo API
using DotNetNuke.Entities.Portals;
using DotNetNuke.Entities.Users;
using DotNetNuke.Security.Membership;
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Net;
using System.Threading;
using System.Web;
using System.Web.Services;

/// <summary>
/// Summary description for SeecApi
/// </summary>
[WebService(Namespace = "http://tempuri.org/")]
[WebServiceBinding(ConformsTo = WsiProfiles.BasicProfile1_1)]
// To allow this Web Service to be called from script, using ASP.NET AJAX, uncomment the following line. 
 [System.Web.Script.Services.ScriptService]
public class SeecApi : System.Web.Services.WebService
{
    [WebMethod]
    public Usuario Login(string usuario, string password)
    {
        Usuario objUser = new Usuario();
        objUser.userId = -1;
        DotNetNuke.Security.Membership.UserLoginStatus status = new DotNetNuke.Security.Membership.UserLoginStatus();
        status = UserLoginStatus.LOGIN_FAILURE;
        var obj = UserController.ValidateUser(0,
            usuario, password,
            "DNN", "secc",
            DotNetNuke.Services.Authentication.AuthenticationLoginBase.GetIPAddress(),
            ref status);

        if (status == UserLoginStatus.LOGIN_SUCCESS || status == UserLoginStatus.LOGIN_SUPERUSER)
        {
            objUser.userId = obj.UserID;
            objUser.displayName = obj.DisplayName;
            return objUser;
        }
        return objUser;
    }

    [WebMethod]
    public string traerMenu()
    {
        seec_dbEntities db = new seec_dbEntities();

        string htmlMenu = "<div data-role='collapsibleset' data-corners='false' data-theme='a' data-content-theme='a' style='margin-top:20px;'>";

        var ejes = db.seec_ejes.Where(x => x.estado == true).ToList();

        foreach (var eje in ejes)
        {
            htmlMenu += "<div data-role='collapsible'>";
            htmlMenu += "<h3>" + eje.descripcion + "</h3>";
            if (eje.seec_materias.Where(x => x.estado == true).ToList().Count > 0)
            {
                htmlMenu += "<div data-role='collapsibleset' data-corners='false' data-theme='a' data-content-theme='a'>";

                foreach (var materia in eje.seec_materias.Where(x => x.estado == true).ToList())
                {
                    htmlMenu += "<div data-role='collapsible'>";
                    htmlMenu += "<h3>" + materia.descripcion + "</h3>";

                    if (materia.seec_tema.Where(x => x.estado == true).ToList().Count > 0)
                    {
                        htmlMenu += "<div data-role='collapsibleset' data-corners='false' data-theme='a' data-content-theme='a'>";

                        foreach (var tema in materia.seec_tema.Where(x => x.estado == true).ToList())
                        {
                            htmlMenu += "<div data-role='collapsible'>";
                            htmlMenu += "<h3>" + tema.descripcion + "</h3>";

                            if (tema.seec_subtema.Where(x => x.estado == true).ToList().Count > 0)
                            {
                                htmlMenu += "<div data-role='collapsibleset' data-corners='false' data-theme='a' data-content-theme='a'>";
                                foreach (var subtema in tema.seec_subtema.Where(x => x.estado == true).ToList())
                                {
                                    string value = eje.descripcion + " ";
                                    value += materia.descripcion + " ";
                                    value += tema.descripcion + " ";
                                    value += subtema.descripcion + " ";

                                    htmlMenu += "<a href='#' class='ui-btn btnGuiaSubtema' id='"+ subtema.id + "' value='" + value + "'>" + subtema.descripcion + "</a>";


                                }
                                htmlMenu += "</div>";
                            }

                            htmlMenu += "</div>";

                        }
                        htmlMenu += "</div>";
                    }

                    htmlMenu += "</div>";

                }
                htmlMenu += "</div>";
            }
            htmlMenu += "</div>";

        }

        return htmlMenu;


    }

    [WebMethod]
    public string traerEjes()
    {
        seec_dbEntities db = new seec_dbEntities();

        string htmlMenu = "<option value=\"-1\">Elija Eje</option>";

        var ejes = db.seec_ejes.Where(x => x.estado == true).ToList();

        foreach (var eje in ejes)
        {
            int numSubTema = 0;
            foreach (var materia in eje.seec_materias.Where(x => x.estado == true).ToList())
            {
                foreach (var tema in materia.seec_tema.Where(x => x.estado == true).ToList())
                {
                    numSubTema += tema.seec_subtema.Count;
                }
            }
            htmlMenu += "<option value=\"" + eje.id + "\" numSubTema=\""+ numSubTema + "\">" + eje.descripcion + "</option>";
        }

        return htmlMenu;
    }

    [WebMethod]
    public string traerEjesPlan()
    {
        seec_dbEntities db = new seec_dbEntities();

        string htmlMenu = "<option value=\"-1\">Elija Eje</option>";

        var ejes = db.seec_ejes.Where(x => x.estado == true).ToList();

        foreach (var eje in ejes)
        {
            
            htmlMenu += "<option value=\"" + eje.id + "\"> " + eje.descripcion + "</option>";
        }

        return htmlMenu;
    }

    [WebMethod]
    public bool GuardarPlan(string tipo, int userId, int ejeId,int numSubtemas,int[] horarios, int[] dias)
    {
        seec_dbEntities db = new seec_dbEntities();
        seec_plan_estudio objPlanEstudio = new seec_plan_estudio();

        List<seec_ejes> ejes = null;
        if (ejeId != -1)
            ejes = db.seec_ejes.Where(x => x.estado == true && x.id == ejeId).ToList();
        else
            ejes = db.seec_ejes.Where(x => x.estado == true).ToList();

        List<seec_plan_estudio_detalle> listaDetalle = new List<seec_plan_estudio_detalle>();

        foreach (var eje in ejes)
        {
            foreach (var materia in eje.seec_materias.Where(x => x.estado == true).ToList())
            {
                foreach (var tema in materia.seec_tema.Where(x => x.estado == true).ToList())
                {
                    numSubtemas += tema.seec_subtema.Count;
                    foreach (var subTema in tema.seec_subtema.Where(x => x.estado == true).ToList())
                    {
                        seec_plan_estudio_detalle objDetalle = new seec_plan_estudio_detalle();
                        //objDetalle.plan_estudio_id = objPlanEstudio.id;
                        objDetalle.estado = "P";
                        objDetalle.sub_tema_id = subTema.id;
                        objDetalle.tiempo_estimado = 0;
                        objDetalle.eje_id = eje.id;
                        objDetalle.fin_eje = false;

                        listaDetalle.Add(objDetalle);
                    }
                }
            }

            var ultimoDetalle = listaDetalle.Where(x => x.eje_id == eje.id).OrderByDescending(x => x.sub_tema_id).FirstOrDefault();
            ultimoDetalle.fin_eje = true;
        }

        objPlanEstudio.usuario_id = userId;
        objPlanEstudio.tipo = tipo;
        if (tipo == "E")
            objPlanEstudio.eje_id = ejeId;

        objPlanEstudio.matutino = false;
        objPlanEstudio.vesterino = false;
        objPlanEstudio.nocturno = false;

        int total_tiempo = 0;

        foreach (var hora in horarios)
        {
            if (hora == 300)
                objPlanEstudio.matutino = true;
            if (hora == 360)
                objPlanEstudio.vesterino = true;
            if (hora == 180)
                objPlanEstudio.nocturno = true;
            total_tiempo += hora;
        }

        string misDias = "";
        foreach (var dia in dias)
        {
            misDias += (misDias != "" ? "|" : "") + dia.ToString();
        }
        objPlanEstudio.dias = misDias;
        objPlanEstudio.estado = "P";
        objPlanEstudio.fecha_ingreso = DateTime.Now;
        objPlanEstudio.avance_minutos = 0;

        decimal tiempo_subtema = (decimal)(total_tiempo * dias.Length * ejes.Count) / numSubtemas;
        decimal sumaTiempo = 0;
        foreach (var ls in listaDetalle)
        {
            ls.tiempo_estimado = tiempo_subtema;
            objPlanEstudio.seec_plan_estudio_detalle.Add(ls);
            sumaTiempo += Math.Round(tiempo_subtema,2);
        }

        db.seec_plan_estudio.Add(objPlanEstudio);
        db.SaveChanges();

        var sm = db.seec_plan_estudio_detalle.Where(x => x.plan_estudio_id == objPlanEstudio.id).Sum(x=>x.tiempo_estimado);
        objPlanEstudio.total_minutos = (int)sm;
        db.SaveChanges();

        return true;
    }

    [WebMethod]
    public PlanEtudio TienePlan(int userId)
    {
        seec_dbEntities db = new seec_dbEntities();
        PlanEtudio objPlan = new PlanEtudio();

        objPlan.id = -1;

        var plan = db.seec_plan_estudio.Where(x => x.usuario_id == userId && x.estado=="P").OrderByDescending(x => x.id)
            .Select(x=> new PlanEtudio{
                id=x.id,
                estado=x.estado,
                matutino = x.matutino,
                vesterino = x.vesterino,
                nocturno = x.nocturno,
                dias = x.dias,
                total_minutos = x.total_minutos,
                avance_minutos = x.seec_plan_estudio_detalle.Where(y=>y.estado=="C").ToList().Count,
                numSubtemas=x.seec_plan_estudio_detalle.Count,
                eje_id = x.eje_id,
                tipo=x.tipo
            })
            .FirstOrDefault();
        if (plan != null)
        {
            objPlan = plan;
            var planEstudio = db.seec_plan_estudio.Where(x => x.id == plan.id).FirstOrDefault();
            var detallePlan = planEstudio.seec_plan_estudio_detalle.Where(x => x.estado == "P").FirstOrDefault();
            string value =  detallePlan.seec_subtema.seec_tema.seec_materias.descripcion + " ";
            value += detallePlan.seec_subtema.seec_tema.descripcion + " ";
            value += detallePlan.seec_subtema.descripcion + " ";
            objPlan.proxSubtemaLnk = value;

            value = "<strong>Materia: </strong>" + detallePlan.seec_subtema.seec_tema.seec_materias.descripcion + " - ";
            value += "<strong>Tema: </strong>" + detallePlan.seec_subtema.seec_tema.descripcion + " - ";
            value += "<strong>SubTema: </strong>" + detallePlan.seec_subtema.descripcion;
            objPlan.proxSubtema = value;

        }
        return objPlan;
    }

    [WebMethod]
    public bool EliminarPlan(int planId)
    {
        seec_dbEntities db = new seec_dbEntities();
        seec_plan_estudio objPlan = db.seec_plan_estudio.Where(x => x.id == planId).FirstOrDefault();
        db.seec_plan_estudio.Remove(objPlan);
        db.SaveChanges();
        return true;
    }


    [WebMethod]
    public bool InsertUpdateToken(int userId,string token)
    {
        seec_dbEntities db = new seec_dbEntities();
        user_token objtoken = db.user_token.Where(x => x.usuario_id == userId).OrderByDescending(x => x.id).FirstOrDefault();
        if(objtoken==null)
        {
            objtoken = new user_token();
            objtoken.usuario_id = userId;
            objtoken.token = token;
            db.user_token.Add(objtoken);

        }
        else
        {
            objtoken.token = token;
        }

        db.SaveChanges();
        
        return true;
    }

    [WebMethod]
    public string GetAlias(string token)
    {
        seec_dbEntities db = new seec_dbEntities();
        string alias = "Device" + (db.user_token.ToList().Count + 1);
        //pushbots
        string HostURI = "http://52.7.167.44/pushseec/setalias.php?alias=" + alias + "&token=" + token;

        HttpWebRequest request = (HttpWebRequest)HttpWebRequest.Create(HostURI);
        request.Method = "GET";
        String test = String.Empty;
        using (HttpWebResponse response = (HttpWebResponse)request.GetResponse())
        {
            Stream dataStream = response.GetResponseStream();
            StreamReader reader = new StreamReader(dataStream);
            test = reader.ReadToEnd();
            reader.Close();
            dataStream.Close();
        }


        return alias;
    }

    [WebMethod]
    public bool EnviarNotificaciones(int opcion)
    {
        seec_dbEntities db = new seec_dbEntities();

        string numDia = ((int)DateTime.Now.DayOfWeek).ToString();
        int minutosAvance = 0;
        List<seec_plan_estudio> planes = new List<seec_plan_estudio>();
        if (opcion == 1) {
            planes = db.seec_plan_estudio.Where(x => x.estado == "P" && x.matutino==true && x.dias.Contains(numDia)).ToList();
            minutosAvance = 300;
        }

        if (opcion == 2)
        {
            planes = db.seec_plan_estudio.Where(x => x.estado == "P" && x.vesterino == true && x.dias.Contains(numDia)).ToList();
            minutosAvance = 360;
        }

        if (opcion == 3)
        {
            planes = db.seec_plan_estudio.Where(x => x.estado == "P" && x.nocturno == true && x.dias.Contains(numDia)).ToList();
            minutosAvance = 180;
        }

        foreach (var plan in planes)
        {
            var detallePlan = plan.seec_plan_estudio_detalle.Where(x => x.estado == "P").FirstOrDefault();

            string url = detallePlan.seec_subtema.seec_tema.seec_materias.seec_ejes.descripcion + " "+ detallePlan.seec_subtema.seec_tema.seec_materias.descripcion + " ";
            url += detallePlan.seec_subtema.seec_tema.descripcion + " ";
            url += detallePlan.seec_subtema.descripcion + " ";

            string msn = "Usted debe estudiar.  Materia: " + detallePlan.seec_subtema.seec_tema.seec_materias.descripcion + " - ";
            msn += "Tema: " + detallePlan.seec_subtema.seec_tema.descripcion + " - ";
            msn += "SubTema: " + detallePlan.seec_subtema.descripcion;

            var alias = db.user_token.Where(x => x.usuario_id == plan.usuario_id).OrderByDescending(x => x.id).FirstOrDefault();

            string HostURI = "http://52.7.167.44/pushseec/example.php?userId="+ alias.token + "&url="+ url + "&mensaje="+msn + "&subtemaid=" + detallePlan.seec_subtema.id;

            HttpWebRequest request = (HttpWebRequest)HttpWebRequest.Create(HostURI);
            request.Method = "GET";
            String test = String.Empty;
            using (HttpWebResponse response = (HttpWebResponse)request.GetResponse())
            {
                Stream dataStream = response.GetResponseStream();
                StreamReader reader = new StreamReader(dataStream);
                test = reader.ReadToEnd();
                reader.Close();
                dataStream.Close();
            }


            Worker workerObject = new Worker();
            workerObject.SetPlan(plan);
            workerObject.SetTiempo(minutosAvance);
            workerObject.SetDb(db);
            Thread workerThread = new Thread(workerObject.DoWork);
            workerThread.Start();

        }

        return true;
    }



    [WebMethod]
    public bool TeminarJornada(int opcion)
    {
        seec_dbEntities db = new seec_dbEntities();

        string numDia = ((int)DateTime.Now.DayOfWeek).ToString();

        List<seec_plan_estudio> planes = new List<seec_plan_estudio>();
        if (opcion == 1)
        {
            planes = db.seec_plan_estudio.Where(x => x.estado == "P" && x.matutino == true && x.dias.Contains(numDia)).ToList();
        }

        if (opcion == 2)
        {
            planes = db.seec_plan_estudio.Where(x => x.estado == "P" && x.vesterino == true && x.dias.Contains(numDia)).ToList();
        }

        if (opcion == 3)
        {
            planes = db.seec_plan_estudio.Where(x => x.estado == "P" && x.nocturno == true && x.dias.Contains(numDia)).ToList();
        }

        foreach (var plan in planes)
        {
           

            if (opcion == 1)
            {
                plan.avance_minutos += 300;
            }

            if (opcion == 2)
            {
                plan.avance_minutos += 360;
            }
            if (opcion == 3)
            {
                plan.avance_minutos += 180;
            }

            decimal countMin = 0;
            foreach (var detalle in plan.seec_plan_estudio_detalle.OrderBy(x=>x.id))
            {
                countMin += (decimal)detalle.tiempo_estimado;
                if (countMin <= plan.avance_minutos && detalle.estado == "P")
                    detalle.estado = "C";
                if (countMin >= plan.avance_minutos)
                    break;
            }

            string testEje = "";

            if ( plan.tipo=="E" && countMin >= plan.total_minutos-5)
            {
                testEje = "" + plan.eje_id;
                plan.estado = "C";
                plan.fecha_termino = DateTime.Now;
            }

            db.SaveChanges();

            if (plan.tipo == "C")
            {
                var testPendintes = plan.seec_plan_estudio_detalle.Where(x => x.estado == "C" && x.fin_eje == true).ToList();
                foreach (var tp in testPendintes)
                {
                    testEje += (testEje != "" ? "|" : "") + tp.eje_id;
                    tp.fin_eje = false;
                }

                if(countMin >= plan.total_minutos - 5)
                {
                    plan.estado = "C";
                    plan.fecha_termino = DateTime.Now;
                    testEje = "-1";
                }
            }

            db.SaveChanges();

            var alias = db.user_token.Where(x => x.usuario_id == plan.usuario_id).OrderByDescending(x => x.id).FirstOrDefault();

            string HostURI = "http://52.7.167.44/pushseec/finalizar.php?alias=" + alias.token + "&test=" + testEje;

            HttpWebRequest request = (HttpWebRequest)HttpWebRequest.Create(HostURI);
            request.Method = "GET";
            String test = String.Empty;
            using (HttpWebResponse response = (HttpWebResponse)request.GetResponse())
            {
                Stream dataStream = response.GetResponseStream();
                StreamReader reader = new StreamReader(dataStream);
                test = reader.ReadToEnd();
                reader.Close();
                dataStream.Close();
            }

        }

        return true;
    }


    [WebMethod]
    public bool TeminarTemas(int opcion)
    {
        seec_dbEntities db = new seec_dbEntities();

        string numDia = ((int)DateTime.Now.DayOfWeek).ToString();

        List<seec_plan_estudio> planes = new List<seec_plan_estudio>();
        if (opcion == 1)
        {
            planes = db.seec_plan_estudio.Where(x => x.estado == "P" && x.matutino == true && x.dias.Contains(numDia)).ToList();
        }

        if (opcion == 2)
        {
            planes = db.seec_plan_estudio.Where(x => x.estado == "P" && x.vesterino == true && x.dias.Contains(numDia)).ToList();
        }

        if (opcion == 3)
        {
            planes = db.seec_plan_estudio.Where(x => x.estado == "P" && x.nocturno == true && x.dias.Contains(numDia)).ToList();
        }

        foreach (var plan in planes)
        {
            var det = plan.seec_plan_estudio_detalle.Where(x => x.estado == "P").OrderBy(x => x.id).FirstOrDefault();
            det.estado = "C";
            plan.avance_minutos += det.tiempo_estimado;
            db.SaveChanges();
        }

        return true;
    }

    

    public SeecApi()
    {

        //Uncomment the following line if using designed components 
        //InitializeComponent(); 
    }

    [WebMethod]
    public string HelloWorld()
    {
        return "Hello World";
    }


    public class Worker
    {
        // This method will be called when the thread is started.
        public void DoWork()
        {
            
            bool continuar = true;

            while (continuar)
            {
                var detAnt = plan.seec_plan_estudio_detalle.Where(x => x.estado == "P").OrderBy(x => x.id).FirstOrDefault();

                System.Threading.Thread.Sleep(Convert.ToInt32(1000 * 60 * detAnt.tiempo_estimado));
                detAnt.estado = "C";
                db.SaveChanges();

                decimal countMin = db.seec_plan_estudio_detalle.Where(x => x.plan_estudio_id == plan.id && x.estado == "C").Sum(x => x.tiempo_estimado) != null ? (decimal)db.seec_plan_estudio_detalle.Where(x => x.plan_estudio_id == plan.id && x.estado == "C").Sum(x => x.tiempo_estimado) : 0;

                if (countMin >= (plan.avance_minutos + tiempoOpcion) - 5)
                {
                    continuar = false;
                }
                else {
                    
                    var detallePlan = plan.seec_plan_estudio_detalle.Where(x => x.estado == "P").OrderBy(x => x.id).FirstOrDefault();

                    string url = detallePlan.seec_subtema.seec_tema.seec_materias.seec_ejes.descripcion + " " + detallePlan.seec_subtema.seec_tema.seec_materias.descripcion + " ";
                    url += detallePlan.seec_subtema.seec_tema.descripcion + " ";
                    url += detallePlan.seec_subtema.descripcion + " ";

                    string msn = "Usted debe estudiar.  Materia: " + detallePlan.seec_subtema.seec_tema.seec_materias.descripcion + " - ";
                    msn += "Tema: " + detallePlan.seec_subtema.seec_tema.descripcion + " - ";
                    msn += "SubTema: " + detallePlan.seec_subtema.descripcion;

                    var alias = db.user_token.Where(x => x.usuario_id == plan.usuario_id).OrderByDescending(x => x.id).FirstOrDefault();

                    string HostURI = "http://52.7.167.44/pushseec/example.php?userId=" + alias.token + "&url=" + url + "&mensaje=" + msn + "&subtemaid=" + detallePlan.seec_subtema.id;

                    HttpWebRequest request = (HttpWebRequest)HttpWebRequest.Create(HostURI);
                    request.Method = "GET";
                    String test = String.Empty;
                    using (HttpWebResponse response = (HttpWebResponse)request.GetResponse())
                    {
                        Stream dataStream = response.GetResponseStream();
                        StreamReader reader = new StreamReader(dataStream);
                        test = reader.ReadToEnd();
                        reader.Close();
                        dataStream.Close();
                    }
                }
            }

        }

        public void SetDb(seec_dbEntities miDb)
        {
            db = miDb;
        }
        public void SetPlan(seec_plan_estudio miPlan)
        {
            plan = miPlan;
        }
        public void SetTiempo(int miTiempoOpcion)
        {
            tiempoOpcion = miTiempoOpcion;
        }
        // Volatile is used as hint to the compiler that this data
        // member will be accessed by multiple threads.
        private seec_plan_estudio plan;
        private int tiempoOpcion;
        private seec_dbEntities db;
    }



    public class Usuario
    {
        public int userId { set; get; }
        public string displayName { set; get; }
        public string rol { set; get; }
    }

    public class misEjes
    {
        public int id { set; get; }
        public string descripcion { set; get; }
        public int numSubTema { set; get; }
    }

    public class PlanEtudio
    {
        public int id { set; get; }
        public string estado { set; get; }
        public bool? matutino { set; get; }
        public bool? vesterino { set; get; }
        public bool? nocturno { set; get; }
        public string dias { set; get; }
        public int? total_minutos { set; get; }
        public int? avance_minutos { set; get; }
        public int numSubtemas { set; get; }
        public int? eje_id { set; get; }
        public string proxSubtema { set; get; }
        public string proxSubtemaLnk { set; get; }
        public string tipo { set; get; }
    }

}

