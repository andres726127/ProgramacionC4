Paquete CLASES

```java
package Clases;

import javax.persistence.*;
import java.util.ArrayList;
import java.util.List;

@Entity
@Table(name = "propietarios")
public class Propietario {
    @Id
    @Column(name = "cedula", nullable = false, length = 10) // verificacion adicional
    private String cedula;

    @Column(name = "nombres", nullable = false)
    private String nombres;

    @Column(name = "apellidos", nullable = false)
    private String apellidos;

    @OneToMany(mappedBy = "propietario", cascade = CascadeType.ALL, fetch = FetchType.LAZY) //represento la relacion que existe entre la clase vehiculo y propietario
    private List<Vehiculo> carros = new ArrayList<>();

    // Constructores
    public Propietario() {}

    public Propietario(String cedula, String nombres, String apellidos) {
        this.cedula = cedula;
        this.nombres = nombres;
        this.apellidos = apellidos;
    }

    // Getters y Setters
    public String getCedula() { return cedula; }

    public String getNombres() {
        return nombres;
    }

    public void setNombres(String nombres) {
        this.nombres = nombres;
    }

    public String getApellidos() {
        return apellidos;
    }

    public void setApellidos(String apellidos) {
        this.apellidos = apellidos;
    }

    public List<Vehiculo> getCarros() {
        return carros;
    }

    public void setCarros(List<Vehiculo> carros) {
        this.carros = carros;
    }


    // Método para agregar carros
    public void agregarCarro(Vehiculo carro) {
        carro.setProp(this);
        this.carros.add(carro);
    }
}

```java
package Clases;

import javax.persistence.*;
import java.util.ArrayList;
import java.util.List;

@Entity
@Table(name = "turnos")
public class Turno {
    @Id
    @Column(name = "idTurn", nullable = false) 
    private int idTurn;

    @Column(name = "Anden", nullable = false)
    private int Anden;

    @Column(name = "dia", nullable = false)
    private int dia;
    
     @Column(name = "hora", nullable = false)
    private int hora;


    
    @ManyToOne(fetch = FetchType.LAZY) // como no existe la operacion Muchos a zero en  javax.persistence.* , entonces utilizo muchos a uno
    @JoinColumn(name = "id_Veh", nullable = false)
    private Vehiculo carro;
    
    
    // Constructores

    public Turno(int Anden, int dia, int hora) {
        this.Anden = Anden;
        this.dia = dia;
        this.hora = hora;
    }

 

    public Turno() {
    }
    
      // Getters y Setters

    public int getIdTurn() {
        return idTurn;
    }

    public void setIdTurn(int idTurn) {
        this.idTurn = idTurn;
    }

    public int getAnden() {
        return Anden;
    }

    public void setAnden(int Anden) {
        this.Anden = Anden;
    }

    public int getDia() {
        return dia;
    }

    public void setDia(int dia) {
        this.dia = dia;
    }

    public int getHora() {
        return hora;
    }

    public void setHora(int hora) {
        this.hora = hora;
    }
 
    

    public Vehiculo getCarro() {
        return carro;
    }

    public void setCarro(Vehiculo carro) {
        this.carro = carro;
    }
    
}
```

```java
package Clases;

import java.util.ArrayList;
import java.util.List;
import javax.persistence.*;

@Entity
@Table(name = "vehiculos")
public class Vehiculo {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "id_Veh")
    private Long id;

    @Column(name = "placa", nullable = false)
    private String placa;

    @Column(name = "marca", nullable = false)
    private String marca;

    @Column(name = "estado", nullable = false)
    private int estado;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "cedula", nullable = false)
    private Propietario prop;

    @OneToMany(mappedBy = "vehiculo", cascade = CascadeType.ALL, fetch = FetchType.LAZY) //represento la relacion que existe entre la clase vehiculo y propietario
    private List<Turno> turnos = new ArrayList<>(); // como no existe la operacion zero  a mucho en  javax.persistence.* , entonces utilizo uno a muchos

    // Constructores
    public Vehiculo() {
    }

    public Vehiculo(String placa, String marca, int estado) {
        this.placa = placa;
        this.marca = marca;
        this.estado = estado;
        this.prop = prop;
    }

    public String getPlaca() {
        return placa;
    }

    public void setPlaca(String placa) {
        this.placa = placa;
    }

    public String getMarca() {
        return marca;
    }

    public void setMarca(String marca) {
        this.marca = marca;
    }



    public void setEstado(int estado) {
        this.estado = estado;
    }

    public int getEstado() {
        return estado;
    }

    public Propietario getProp() {
        return prop;
    }

    // Getters y Setters
    public void setProp(Propietario prop) {
        this.prop = prop;
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    // Método para agregar turnos
    public void agregarTurno(Turno turno) {
        turno.setCarro(this);
        this.turnos.add(turno);
    }

    public List<Turno> getTurnos() {
        return turnos;
    }

}

```

Paquete LOGICA
```java
package Logica;

import Clases.Propietario;
import Clases.Vehiculo;
import Clases.Turno;

import javax.persistence.EntityManager;
import javax.persistence.EntityManagerFactory;
import javax.persistence.Persistence;
import javax.persistence.TypedQuery;
import java.util.List;
// import java.time.LocalDate;

public class PropServicio {

    private final EntityManagerFactory emf;

    public PropServicio() {
        this.emf = Persistence.createEntityManagerFactory("test1PU");
    }

    // Insertar prop
    public void insertarPropietario(Propietario prop) throws Exception {
        if (!validarCedula(prop.getCedula())) {
            throw new Exception("C�dula no v�lida.");
        }
        EntityManager em = emf.createEntityManager();
        try {
            em.getTransaction().begin();
            em.persist(prop);
            em.getTransaction().commit();
        } catch (Exception e) {
            em.getTransaction().rollback();
            throw new Exception("Error al insertar prop: " + e.getMessage());
        } finally {
            em.close();
        }
    }

    // Insertar vehiculo para un prop
    public void insertarVehiculo(String cedula, Vehiculo carro) throws Exception {
        EntityManager em = emf.createEntityManager();
        try {
            em.getTransaction().begin();
            Propietario prop = em.find(Propietario.class, cedula);
            if (prop == null) {
                throw new Exception("Propietario no encontrado.");
            }
            /* No hay necesidad de hacer ningun chequeo if (carro.isEsActual()) {
                // Desactivar otras direcciones
                for (Direccion d : prop.getDirecciones()) {
                    d.setEsActual(false);
                    em.merge(d);
                }
            }*/
            prop.agregarCarro(carro);
            em.merge(prop);
            em.getTransaction().commit();
        } catch (Exception e) {
            em.getTransaction().rollback();
            throw new Exception("Error al insertar carro: " + e.getMessage());
        } finally {
            em.close();
        }
    }

    // Insertar turnos para un vehiculo
    public void insertarTurno(String id_Veh, Turno turno, int dia) throws Exception {
        EntityManager em = emf.createEntityManager();
        try {
            em.getTransaction().begin();
            Vehiculo carro = em.find(Vehiculo.class, id_Veh);
            if (carro == null) {
                throw new Exception("Turno no encontrado.");
            }

            // Verificacioin
            // Activar solo 1 turno por dia
            for (Turno t : carro.getTurnos()) {
                if (dia != t.getDia() || carro.getEstado()==0) {
                    carro.agregarTurno(turno);
                    carro.setEstado(1); // no se puede agregar mas turnos al mismo carro, a menos de que el dia cambie
                }
            }
            em.merge(turno);
            em.getTransaction().commit();

        } catch (Exception e) {
            em.getTransaction().rollback();
            throw new Exception("Error al insertar turno: " + e.getMessage());
        } finally {
            em.close();
        }
    }

    // Cambiar direcci�n actual
    /*  public void cambiarDireccionActual(String cedula, Long idDireccion) throws Exception {
        EntityManager em = emf.createEntityManager();
        try {
            em.getTransaction().begin();
            Propietario prop = em.find(Propietario.class, cedula);
            if (prop == null) {
                throw new Exception("Propietario no encontrado.");
            }
            for (Direccion d : prop.getDirecciones()) {
                if (d.getId().equals(idDireccion)) {
                    d.setEsActual(true);
                } else {
                    d.setEsActual(false);
                }
                em.merge(d);
            }
            em.getTransaction().commit();
        } catch (Exception e) {
            em.getTransaction().rollback();
            throw new Exception("Error al cambiar direcci�n actual: " + e.getMessage());
        } finally {
            em.close();
        }
    }*/

    // Buscar prop por c�dula
    public Propietario buscarPropPorCedula(String cedula) throws Exception {
        EntityManager em = emf.createEntityManager();
        try {
            Propietario prop = em.find(Propietario.class, cedula);
            if (prop == null) {
                throw new Exception("Propietario no encontrado.");
            }
            return prop;
        } finally {
            em.close();
        }
    }

    // Buscar clientes por nombre
    public List<Propietario> buscarPropPorNombre(String nombre) {
        EntityManager em = emf.createEntityManager();
        try {
            TypedQuery<Propietario> query = em.createQuery(
                "SELECT c FROM Propietario c WHERE LOWER(c.nombres) LIKE LOWER(:nombre)", Propietario.class);
            query.setParameter("nombre", "%" + nombre + "%");
            return query.getResultList();
        } finally {
            em.close();
        }
    }
    // Validar c�dula (ejemplo simple, puedes mejorar seg�n reglas locales)
    private boolean validarCedula(String cedula) {
        return cedula != null && cedula.matches("\\d{10}"); //regex pa que hallan justo 10 enteros
    }

    // Cerrar EntityManagerFactory
    public void cerrar() {
        if (emf.isOpen()) {
            emf.close();
        }
    }
}

```
Paquete PRESENTACION
```java
package Presentacion;

import Clases.*;
import Logica.PropServicio;
import java.util.List;
import javax.swing.*;
import java.time.LocalDate;

public class AgregarTurno extends JFrame {

    private JTextField fieldCedula, fieldCalle1, fieldCalle2, fieldCalle3;
    private JCheckBox checkActual;
    private JLabel lblUsuario;
    private JButton btnBuscar, btnAgregar;
    private final PropServicio servicio;
    private Propietario prop;
    private Vehiculo carro;

    public AgregarTurno() {
        servicio = new PropServicio();
        initComponents();
        setTitle("Agregar Turnos");
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setResizable(false);
        setLocationRelativeTo(null);
    }

    private void initComponents() {
        setLayout(null);

        JLabel lblCedula = new JLabel("Cédula:");
        lblCedula.setBounds(20, 20, 100, 25);
        add(lblCedula);

        fieldCedula = new JTextField();
        fieldCedula.setBounds(120, 20, 200, 25);
        add(fieldCedula);

        btnBuscar = new JButton("Buscar");
        btnBuscar.setBounds(330, 20, 100, 25);
        btnBuscar.addActionListener(e -> buscarProp());
        add(btnBuscar);

        lblUsuario = new JLabel("Propietario: No seleccionado");
        lblUsuario.setBounds(20, 50, 400, 25);
        add(lblUsuario);

        JLabel lblCalle1 = new JLabel("Turno 1:");
        lblCalle1.setBounds(20, 80, 100, 25);
        add(lblCalle1);

        fieldCalle1 = new JTextField();
        fieldCalle1.setBounds(120, 80, 200, 25);
        add(fieldCalle1);

        JLabel lblCalle2 = new JLabel("Carro 2:");
        lblCalle2.setBounds(20, 110, 100, 25);
        add(lblCalle2);

        fieldCalle3 = new JTextField();
        fieldCalle3.setBounds(120, 80, 200, 25);
        add(fieldCalle3);

        JLabel lblCalle3 = new JLabel("Carro 3:");
        lblCalle3.setBounds(20, 110, 100, 25);
        add(lblCalle3);

        fieldCalle2 = new JTextField();
        fieldCalle2.setBounds(120, 110, 200, 25);
        add(fieldCalle2);

        checkActual = new JCheckBox("¿Su carro ha sido registrado?");
        checkActual.setBounds(120, 140, 200, 25);
        add(checkActual);

        btnAgregar = new JButton("Agregar Turno");
        btnAgregar.setBounds(120, 170, 150, 30);
        btnAgregar.addActionListener(e -> enviarTurno());
        add(btnAgregar);

        setSize(450, 250);
    }

    private void buscarProp() {
        try {
            String cedula = fieldCedula.getText().trim();
            if (cedula.isEmpty()) {
                JOptionPane.showMessageDialog(this, "Ingrese una cédula.", "Error", JOptionPane.ERROR_MESSAGE);
                return;
            }
            prop = servicio.buscarPropPorCedula(cedula);
            lblUsuario.setText("Usuario: " + prop.getNombres() + " " + prop.getApellidos());
        } catch (Exception e) {
            JOptionPane.showMessageDialog(this, e.getMessage(), "Error", JOptionPane.ERROR_MESSAGE);
            prop = null;
            lblUsuario.setText("Usuario: No seleccionado");
        }
    }

    private void enviarTurno() {
        try {
            if (carro == null) {
                JOptionPane.showMessageDialog(this, "Seleccione un propietario válido.", "Error", JOptionPane.ERROR_MESSAGE);
                return;
            }
            int calle1 = Integer.parseInt(fieldCalle1.getText().trim());
            int calle2 = Integer.parseInt(fieldCalle2.getText().trim());
            int esActual = Integer.parseInt(checkActual.getText().trim());

            if (calle2 == 0 || esActual == 0) {
                JOptionPane.showMessageDialog(this, "El diaa y hora no pueden ser 0.", "Error", JOptionPane.ERROR_MESSAGE);
                return;
            }

            Turno t = new Turno(calle1, calle2, esActual);
            servicio.insertarTurno(carro.getId().toString(), t, calle2);
            JOptionPane.showMessageDialog(this, "Dirección agregada con éxito.", "Éxito", JOptionPane.INFORMATION_MESSAGE);
            limpiarCampos();
        } catch (Exception e) {
            JOptionPane.showMessageDialog(this, e.getMessage(), "Error", JOptionPane.ERROR_MESSAGE);
        }
    }

    private void limpiarCampos() {
        fieldCalle1.setText("");
        fieldCalle2.setText("");
        checkActual.setSelected(false);
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> new AgregarTurno().setVisible(true));
    }

}

```

```java
package Presentacion;

import Clases.Propietario;
import Clases.Vehiculo;
import Clases.Turno;
import Logica.PropServicio;

import javax.swing.*;

public class AgregarVehiculo extends JFrame {

    private JTextField fieldCedula, fieldCalle1, fieldCalle2, fieldCalle3;
    private JCheckBox checkActual;
    private JLabel lblUsuario;
    private JButton btnBuscar, btnAgregar;
    private final PropServicio servicio;
    private Propietario prop;

    public AgregarVehiculo() {
        servicio = new PropServicio();
        initComponents();
        setTitle("Agregar Dirección");
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setResizable(false);
        setLocationRelativeTo(null);
    }

    private void initComponents() {
        setLayout(null);

        JLabel lblCedula = new JLabel("Cédula:");
        lblCedula.setBounds(20, 20, 100, 25);
        add(lblCedula);

        fieldCedula = new JTextField();
        fieldCedula.setBounds(120, 20, 200, 25);
        add(fieldCedula);

        btnBuscar = new JButton("Buscar");
        btnBuscar.setBounds(330, 20, 100, 25);
        btnBuscar.addActionListener(e -> buscarPropietario());
        add(btnBuscar);

        lblUsuario = new JLabel("Propietario: No seleccionado");
        lblUsuario.setBounds(20, 50, 400, 25);
        add(lblUsuario);

        JLabel lblCalle1 = new JLabel("Carro 1:");
        lblCalle1.setBounds(20, 80, 100, 25);
        add(lblCalle1);

        fieldCalle1 = new JTextField();
        fieldCalle1.setBounds(120, 80, 200, 25);
        add(fieldCalle1);

        JLabel lblCalle2 = new JLabel("Carro 2:");
        lblCalle2.setBounds(20, 110, 100, 25);
        add(lblCalle2);
        
        fieldCalle3 = new JTextField();
        fieldCalle3.setBounds(120, 80, 200, 25);
        add(fieldCalle3);

        JLabel lblCalle3 = new JLabel("Carro 3:");
        lblCalle3.setBounds(20, 110, 100, 25);
        add(lblCalle3);

        fieldCalle2 = new JTextField();
        fieldCalle2.setBounds(120, 110, 200, 25);
        add(fieldCalle2);

        checkActual = new JCheckBox("¿Su carro ha sido registrado?");
        checkActual.setBounds(120, 140, 200, 25);
        add(checkActual);

        btnAgregar = new JButton("Agregar Dirección");
        btnAgregar.setBounds(120, 170, 150, 30);
        btnAgregar.addActionListener(e -> agregarVehiculo());
        add(btnAgregar);

        setSize(450, 250);
    }

    private void buscarPropietario() {
        try {
            String cedula = fieldCedula.getText().trim();
            if (cedula.isEmpty()) {
                JOptionPane.showMessageDialog(this, "Ingrese una cédula.", "Error", JOptionPane.ERROR_MESSAGE);
                return;
            }
            prop = servicio.buscarPropPorCedula(cedula);
            lblUsuario.setText("Usuario: " + prop.getNombres() + " " + prop.getApellidos());
        } catch (Exception e) {
            JOptionPane.showMessageDialog(this, e.getMessage(), "Error", JOptionPane.ERROR_MESSAGE);
            prop = null;
            lblUsuario.setText("Usuario: No seleccionado");
        }
    }

    private void agregarVehiculo() {
        try {
            if (prop == null) {
                JOptionPane.showMessageDialog(this, "Seleccione un propietario válido.", "Error", JOptionPane.ERROR_MESSAGE);
                return;
            }
            String calle1 = fieldCalle1.getText().trim();
            String calle2 = fieldCalle2.getText().trim();
            int esActual = Integer.parseInt(checkActual.getText().trim());

            if (calle1.isEmpty() || calle2.isEmpty()) {
                JOptionPane.showMessageDialog(this, "Las calles no pueden estar vacías.", "Error", JOptionPane.ERROR_MESSAGE);
                return;
            }

            Vehiculo direccion = new Vehiculo(calle1, calle2, esActual);
            servicio.insertarVehiculo(prop.getCedula(), direccion);
            JOptionPane.showMessageDialog(this, "Dirección agregada con éxito.", "Éxito", JOptionPane.INFORMATION_MESSAGE);
            limpiarCampos();
        } catch (Exception e) {
            JOptionPane.showMessageDialog(this, e.getMessage(), "Error", JOptionPane.ERROR_MESSAGE);
        }
    }

    private void limpiarCampos() {
        fieldCalle1.setText("");
        fieldCalle2.setText("");
        checkActual.setSelected(false);
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> new AgregarVehiculo().setVisible(true));
    }
}

```

```java
package Presentacion;

import Clases.Propietario;
import Logica.PropServicio;

import javax.swing.*;
import java.util.List;

public class BusquedaPropietario extends JFrame {
    private JTextField fieldNombre;
    private JTextArea textAreaResultados;
    private JButton btnBuscar, btnVolver, btnEditarDirecciones;
    private final PropServicio servicio;

    public BusquedaPropietario() {
        servicio = new PropServicio();
        initComponents();
        setTitle("Buscar Cliente");
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setResizable(false);
        setLocationRelativeTo(null);
    }

    private void initComponents() {
        setLayout(null);

        JLabel lblNombre = new JLabel("Nombre del Cliente:");
        lblNombre.setBounds(20, 20, 150, 25);
        add(lblNombre);

        fieldNombre = new JTextField();
        fieldNombre.setBounds(170, 20, 200, 25);
        add(fieldNombre);

        btnBuscar = new JButton("Buscar");
        btnBuscar.setBounds(380, 20, 100, 25);
        btnBuscar.addActionListener(e -> buscarProp());
        add(btnBuscar);

        textAreaResultados = new JTextArea();
        textAreaResultados.setEditable(false);
        JScrollPane scrollPane = new JScrollPane(textAreaResultados);
        scrollPane.setBounds(20, 50, 460, 200);
        add(scrollPane);

        btnVolver = new JButton("Volver");
        btnVolver.setBounds(20, 260, 100, 25);
        btnVolver.addActionListener(e -> volver());
        add(btnVolver);

        btnEditarDirecciones = new JButton("Editar Direcciones");
        btnEditarDirecciones.setBounds(360, 260, 120, 25);
        btnEditarDirecciones.addActionListener(e -> abrirEditarDirecciones());
        add(btnEditarDirecciones);

        setSize(500, 320);
    }

    private void buscarProp() {
        try {
            String nombre = fieldNombre.getText().trim();
            if (nombre.isEmpty()) {
                JOptionPane.showMessageDialog(this, "Ingrese un nombre.", "Error", JOptionPane.ERROR_MESSAGE);
                return;
            }
            List<Propietario> props = servicio.buscarPropPorNombre(nombre);
            if (props.isEmpty()) {
                textAreaResultados.setText("No se encontraron propietarios.");
                return;
            }
            StringBuilder texto = new StringBuilder();
            for (Propietario c : props) {
                texto.append("Cédula: ").append(c.getCedula())
                     .append("\nNombres: ").append(c.getNombres())
                     .append("\nApellidos: ").append(c.getApellidos())
                     .append("\n------------------------\n");
            }
            textAreaResultados.setText(texto.toString());
        } catch (Exception e) {
            JOptionPane.showMessageDialog(this, e.getMessage(), "Error", JOptionPane.ERROR_MESSAGE);
        }
    }

    private void volver() {
        new RegistroPropietario().setVisible(true);
        this.dispose();
    }

    private void abrirEditarDirecciones() {
        new ModificarVehiculo().setVisible(true);
        this.dispose();
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> new BusquedaPropietario().setVisible(true));
    }
}
```

```java
package Presentacion;


import Clases.*;
import Logica.PropServicio;

import javax.swing.*;
import java.util.List;

public class ModificarVehiculo extends JFrame {
    private JTextField fieldCedula, fieldIdDireccion;
    private JTextArea textAreaDirecciones;
    private JButton btnConsultar, btnCambiar, btnTurno;
    private final PropServicio servicio;
    private Propietario cliente;

    public ModificarVehiculo() {
        servicio = new PropServicio();
        initComponents();
        setTitle("Agregar Vehiculo Actual");
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setResizable(false);
        setLocationRelativeTo(null);
    }

    private void initComponents() {
        setLayout(null);

        JLabel lblCedula = new JLabel("Cédula del Cliente:");
        lblCedula.setBounds(20, 20, 150, 25);
        add(lblCedula);

        fieldCedula = new JTextField();
        fieldCedula.setBounds(170, 20, 200, 25);
        add(fieldCedula);

        btnConsultar = new JButton("Consultar Vehiculos");
        btnConsultar.setBounds(380, 20, 150, 25);
        btnConsultar.addActionListener(e -> consultarDirecciones());
        add(btnConsultar);

        textAreaDirecciones = new JTextArea();
        textAreaDirecciones.setEditable(false);
        JScrollPane scrollPane = new JScrollPane(textAreaDirecciones);
        scrollPane.setBounds(20, 50, 510, 200);
        add(scrollPane);

        JLabel lblIdDireccion = new JLabel("ID de Vehiculo:");
        lblIdDireccion.setBounds(20, 260, 150, 25);
        add(lblIdDireccion);

        fieldIdDireccion = new JTextField();
        fieldIdDireccion.setBounds(170, 260, 200, 25);
        add(fieldIdDireccion);

        btnCambiar = new JButton("Lo ha registrado antes?");
        btnCambiar.setBounds(380, 260, 150, 25);
        btnCambiar.addActionListener(e -> cambiarDireccionActual());
        add(btnCambiar);
        
        btnTurno = new JButton("Ir a Turnos");
        btnTurno.setBounds(280, 200, 150, 30);
        btnTurno.addActionListener(e -> abrirTurnos());
        add(btnTurno);

        setSize(550, 350);
    }

    private void consultarDirecciones() {
        try {
            String cedula = fieldCedula.getText().trim();
            if (cedula.isEmpty()) {
                JOptionPane.showMessageDialog(this, "Ingrese una cédula.", "Error", JOptionPane.ERROR_MESSAGE);
                return;
            }
            cliente = servicio.buscarPropPorCedula(cedula);
            List<Vehiculo> carros = cliente.getCarros();
            if (carros.isEmpty()) {
                textAreaDirecciones.setText("No se encontraron direcciones.");
                cliente = null;
                return;
            }
            StringBuilder texto = new StringBuilder();
            for (Vehiculo d : carros) {
                texto.append("ID: ").append(d.getId())
                     .append("\nCalle 1: ").append(d.getPlaca())
                     .append("\nCalle 2: ").append(d.getMarca())
                     .append("\nActual: ").append(d.getEstado()==1 ? "Sí" : "No") // if ternario
                     .append("\n------------------------\n");
            }
            textAreaDirecciones.setText(texto.toString());
        } catch (Exception e) {
            JOptionPane.showMessageDialog(this, e.getMessage(), "Error", JOptionPane.ERROR_MESSAGE);
            cliente = null;
            textAreaDirecciones.setText("");
        }
    }

    private void cambiarDireccionActual() {
        try {
            if (cliente == null) {
                JOptionPane.showMessageDialog(this, "Consulte un propietario válido primero.", "Error", JOptionPane.ERROR_MESSAGE);
                return;
            }
            String idStr = fieldIdDireccion.getText().trim();
            if (idStr.isEmpty()) {
                JOptionPane.showMessageDialog(this, "Ingrese el ID de el carro.", "Error", JOptionPane.ERROR_MESSAGE);
                return;
            }
            Long idDireccion = Long.parseLong(idStr);
            //servicio.cambiarDireccionActual(cliente.getCedula(), idDireccion);
            JOptionPane.showMessageDialog(this, "Carro actualizada con éxito.", "Éxito", JOptionPane.INFORMATION_MESSAGE);
            consultarDirecciones(); // Refrescar la lista
        } catch (NumberFormatException e) {
            JOptionPane.showMessageDialog(this, "El ID debe ser un número válido.", "Error", JOptionPane.ERROR_MESSAGE);
        } catch (Exception e) {
            JOptionPane.showMessageDialog(this, e.getMessage(), "Error", JOptionPane.ERROR_MESSAGE);
        }
    }
    
        private void abrirTurnos() {
        new AgregarTurno().setVisible(true);
        this.dispose();
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> new ModificarVehiculo().setVisible(true));
    }
}
```

```java
package Presentacion;

import Clases.Propietario;
import Clases.Vehiculo;
import Logica.PropServicio;

import javax.swing.*;
import java.util.ArrayList;

public class RegistroPropietario extends JFrame {

    private JTextField fieldCedula, fieldNombres, fieldApellidos, fieldCalle1, fieldCalle2;
    private JTextField checkActual;
    private JButton btnGuardar, btnBuscar;
    private final PropServicio servicio;

    public RegistroPropietario() {
        servicio = new PropServicio();
        initComponents();
        setTitle("Registro de Cliente");
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setResizable(false);
        setLocationRelativeTo(null);
    }

    private void initComponents() {
        setLayout(null);

        JLabel lblCedula = new JLabel("Cédula:");
        lblCedula.setBounds(20, 20, 100, 25);
        add(lblCedula);

        fieldCedula = new JTextField();
        fieldCedula.setBounds(120, 20, 200, 25);
        add(fieldCedula);

        JLabel lblNombres = new JLabel("Nombres:");
        lblNombres.setBounds(20, 50, 100, 25);
        add(lblNombres);

        fieldNombres = new JTextField();
        fieldNombres.setBounds(120, 50, 200, 25);
        add(fieldNombres);

        JLabel lblApellidos = new JLabel("Apellidos:");
        lblApellidos.setBounds(20, 80, 100, 25);
        add(lblApellidos);

        fieldApellidos = new JTextField();
        fieldApellidos.setBounds(120, 80, 200, 25);
        add(fieldApellidos);

        JLabel lblCalle1 = new JLabel("Carro 1:");
        lblCalle1.setBounds(20, 110, 100, 25);
        add(lblCalle1);

        fieldCalle1 = new JTextField();
        fieldCalle1.setBounds(120, 110, 200, 25);
        add(fieldCalle1);

        JLabel lblCalle2 = new JLabel("Carro 2:");
        lblCalle2.setBounds(20, 140, 100, 25);
        add(lblCalle2);

        fieldCalle2 = new JTextField();
        fieldCalle2.setBounds(120, 140, 200, 25);
        add(fieldCalle2);

        JLabel lblEstado = new JLabel("¿Ha registrado el vehiculo antes?");
        lblEstado.setBounds(20, 170, 200, 25);
        add(lblEstado);

        checkActual = new JTextField();
        checkActual.setBounds(210, 170, 140, 25);
        add(checkActual);

        btnGuardar = new JButton("Guardar Cliente");
        btnGuardar.setBounds(120, 200, 150, 30);
        btnGuardar.addActionListener(e -> guardarCliente());
        add(btnGuardar);

        btnBuscar = new JButton("Buscar Cliente");
        btnBuscar.setBounds(280, 200, 150, 30);
        btnBuscar.addActionListener(e -> abrirBusqueda());
        add(btnBuscar);

        setSize(450, 300);
    }

    private void guardarCliente() {
        try {
            String cedula = fieldCedula.getText().trim();
            String nombres = fieldNombres.getText().trim();
            String apellidos = fieldApellidos.getText().trim();
            String placa = fieldCalle1.getText().trim();
            String marca = fieldCalle2.getText().trim();
            int estado = Integer.parseInt(checkActual.getText().trim());

            if (cedula.isEmpty() || nombres.isEmpty() || apellidos.isEmpty() || marca.isEmpty() || placa.isEmpty()) {
                JOptionPane.showMessageDialog(this, "Todos los campos son obligatorios.", "Error", JOptionPane.ERROR_MESSAGE);
                return;
            }

            Propietario cliente = new Propietario(cedula, nombres, apellidos);
            Vehiculo carro = new Vehiculo(placa, marca, estado); // aqui puede haber quiza un problema
            cliente.agregarCarro(carro);

            servicio.insertarPropietario(cliente);
            JOptionPane.showMessageDialog(this, "Propietario registrado exitosamente..", "Éxito", JOptionPane.INFORMATION_MESSAGE);
            limpiarCampos();
        } catch (Exception e) {
            JOptionPane.showMessageDialog(this, e.getMessage(), "Error", JOptionPane.ERROR_MESSAGE);
        }
    }

    private void abrirBusqueda() {
        new BusquedaPropietario().setVisible(true);
        this.dispose();
    }

    private void limpiarCampos() {
        fieldCedula.setText("");
        fieldNombres.setText("");
        fieldApellidos.setText("");
        fieldCalle1.setText("");
        fieldCalle2.setText("");
        checkActual.setText("");
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> new RegistroPropietario().setVisible(true));
    }
}

```
